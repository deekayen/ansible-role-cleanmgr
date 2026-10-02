# deekayen.cleanmgr

[![CI](https://github.com/deekayen/ansible-role-cleanmgr/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-cleanmgr/actions/workflows/ci.yml) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-deekayen.cleanmgr-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/deekayen/cleanmgr/) [![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue)

An Ansible role that runs Windows Disk Cleanup (`CleanMgr.exe`) unattended with a chosen set of cleanup handlers. If `cleanmgr.exe` is missing, it first installs the Desktop Experience feature, which provides it.

The role selects handlers by writing `StateFlags<NNNN>` DWORD values of `2` under `HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\VolumeCaches\<handler>`, where `<NNNN>` is `cleanmgr_sagerun`, and removes the value for handlers that are turned off. It then starts `CleanMgr.exe /sagerun:<n>` hidden and waits for it to exit. Microsoft documents the `StateFlags` format in [Automating Disk Cleanup tool in Windows](https://learn.microsoft.com/en-us/troubleshoot/windows-server/backup-and-storage/automating-disk-cleanup-tool).

## Requirements

- ansible-core 2.15 or newer on the controller.
- The `ansible.windows` collection: `ansible-galaxy collection install ansible.windows`.
- A WinRM or SSH connection to the target with administrative rights. The role writes under `HKLM`, can install a Windows feature, and can reboot the host.

## Supported platforms

| Platform | Versions |
| --- | --- |
| Windows | 2016, 2019, 2022 |

CI lints the role and runs `ansible-playbook --syntax-check`; it does not apply the role to a Windows host.

## Installation

From Ansible Galaxy:

```bash
ansible-galaxy role install deekayen.cleanmgr
ansible-galaxy collection install ansible.windows
```

Or pin it in `requirements.yml`:

```yaml
---
roles:
  - name: deekayen.cleanmgr
    src: https://github.com/deekayen/ansible-role-cleanmgr.git
    scm: git
    version: main

collections:
  - name: ansible.windows
```

```bash
ansible-galaxy install -r requirements.yml
```

## Role variables

`meta/argument_specs.yml` validates types before the tasks run.

### General

| Variable | Default | Description |
| --- | --- | --- |
| `cleanmgr_install_reboot` | `true` | Allow reboots during the Desktop Experience install: one to retry after a `FailedRestartRequired` result, and one when the install reports a reboot is required. |
| `cleanmgr_cleanmgr_reboot` | `false` | Reboot after `CleanMgr.exe` runs. `CleanMgr.exe` gives no signal that a reboot is needed, so this reboots every time the cleanup runs or never. |
| `cleanmgr_sagerun` | `"0001"` | Disk Cleanup profile number. Must be a four-digit string; the role asserts this. It forms the `StateFlags<NNNN>` value name and, as an integer, the `/sagerun:` argument. |
| `cleanmgr_install_path` | `C:/Windows/System32/cleanmgr.exe` | Path checked to decide whether Desktop Experience needs installing. Must start with a drive letter; the role asserts this. |

### Cleanup handlers

Each variable is a boolean. `true` writes `StateFlags<NNNN>=2` under the listed `VolumeCaches` subkey; `false` removes that value.

| Variable | Default | `VolumeCaches` subkey |
| --- | --- | --- |
| `cleanmgr_active_setup_temp_folders` | `false` | `Active Setup Temp Folders` |
| `cleanmgr_branchcache` | `false` | `BranchCache` |
| `cleanmgr_downloaded_program_files` | `true` | `Downloaded Program Files` |
| `cleanmgr_internet_cache_files` | `true` | `Internet Cache Files` |
| `cleanmgr_memory_dump_files` | `false` | `Memory Dump Files` |
| `cleanmgr_old_chkdsk_files` | `false` | `Old ChkDsk Files` |
| `cleanmgr_previous_installations` | `false` | `Previous Installations` |
| `cleanmgr_recycle_bin` | `false` | `Recycle Bin` |
| `cleanmgr_service_pack_cleanup` | `false` | `Service Pack Cleanup` |
| `cleanmgr_setup_log_files` | `false` | `Setup Log Files` |
| `cleanmgr_system_error_memory_dump_files` | `false` | `System error memory dump files` |
| `cleanmgr_system_error_minidump_files` | `false` | `System error minidump files` |
| `cleanmgr_temporary_files` | `false` | `Temporary Files` |
| `cleanmgr_temporary_setup_files` | `false` | `Temporary Setup Files` |
| `cleanmgr_thumbnail_cache` | `false` | `Thumbnail Cache` |
| `cleanmgr_upgrade_discarded_files` | `false` | `Upgrade Discarded Files` |
| `cleanmgr_user_file_versions` | `false` | `User file versions` |
| `cleanmgr_windows_defender` | `false` | `Windows Defender` |
| `cleanmgr_windows_error_reporting_archive_files` | `false` | `Windows Error Reporting Archive Files` |
| `cleanmgr_windows_error_reporting_queue_files` | `false` | `Windows Error Reporting Queue Files` |
| `cleanmgr_windows_error_reporting_system_archive_files` | `false` | `Windows Error Reporting System Archive Files` |
| `cleanmgr_windows_error_reporting_system_queue_files` | `false` | `Windows Error Reporting System Queue Files` |
| `cleanmgr_windows_esd_installation_files` | `false` | `Windows ESD installation files` |
| `cleanmgr_update_cleanup` | `true` | `Update Cleanup` |
| `cleanmgr_windows_upgrade_log_files` | `false` | `Windows Upgrade Log Files` |

## Behavior

- The CleanMgr task has `creates: '%windir%\Logs\CBS\DeepClean.log'`. A comment in `tasks/main.yml` says CleanMgr leaves that file behind, so once it exists the cleanup is skipped on every later run. Delete the file to run the cleanup again.
- The registry tasks run on every play, so changing a handler variable updates the profile even when the cleanup itself is skipped.
- `Start-Process` runs without `-PassThru`, so the task's return code reflects the PowerShell command, not CleanMgr's exit code. A return code of `0` marks the task changed and notifies the reboot handler.
- `Start-Process` launches `CleanMgr.exe` from the system `PATH`, not from `cleanmgr_install_path`.
- If the Desktop Experience install returns `FailedRestartRequired` and `cleanmgr_install_reboot` is `false`, the role does not fail. It continues to the registry and CleanMgr tasks without retrying the install.

## Dependencies

None. The `ansible.windows` collection is a requirement, not a role dependency.

## Example playbook

Clear temporary files and the Recycle Bin along with the default handlers, using profile 0042:

```yaml
---
- name: Run Disk Cleanup on Windows file servers.
  hosts: windows_file_servers

  vars:
    cleanmgr_sagerun: "0042"
    cleanmgr_temporary_files: true
    cleanmgr_recycle_bin: true
    cleanmgr_install_reboot: false

  roles:
    - deekayen.cleanmgr
```

## Known issues

- The Desktop Experience branch in `tasks/main.yml:14` through `tasks/main.yml:56` only matters on Windows Server releases older than 2016. Microsoft's [Disk Cleanup on Windows Server](https://learn.microsoft.com/en-us/windows-server/storage/file-server/disk-cleanup) page says `cleanmgr.exe` is present by default on Windows Server 2016 and 2019 and needs Desktop Experience only on earlier versions, and `meta/main.yml` lists 2016 and later.

## Development

CI runs on every push to `main` and every pull request (see `.github/workflows/ci.yml`). It installs `ansible.windows`, runs `ansible-lint --profile production`, and syntax-checks `tests/test.yml` with both reboot variables set to `false`. To run the same checks locally:

```bash
pip3 install ansible-lint
ansible-galaxy collection install ansible.windows
ansible-lint --profile production
mkdir -p .ansible/roles && ln -sfn "$PWD" .ansible/roles/deekayen.cleanmgr
ANSIBLE_ROLES_PATH=.ansible/roles ansible-playbook --syntax-check tests/test.yml -i tests/inventory
```

The repository also has a `.pre-commit-config.yaml`; run `pre-commit run --all-files` before pushing.

### Repository layout

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Desktop Experience install and reboots, `StateFlags` registry values, and the CleanMgr run. |
| `tasks/assert.yml` | Input validation, tagged `always`. |
| `handlers/main.yml` | Reboot after the cleanup when `cleanmgr_cleanmgr_reboot` is `true`. |
| `defaults/main.yml` | Every user-facing variable. |
| `meta/argument_specs.yml` | Type validation for the variables. |
| `tests/` | Syntax-check playbook and inventory used by CI. |

## Releases

Pushing a git tag runs `.github/workflows/release.yml`, which imports the tagged commit into Ansible Galaxy as `deekayen.cleanmgr`. The import needs a `GALAXY_API_KEY` repository or organization secret.

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Author

[David Norman](https://github.com/deekayen). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
