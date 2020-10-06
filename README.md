[![Build Status](https://travis-ci.org/deekayen/ansible-role-cleanmgr.svg?branch=main)](https://travis-ci.org/deekayen/ansible-role-cleanmgr) [![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive)

Disk Cleanup
============

Install the Desktop Experience feature and run Disk Cleanup on Microsoft Windows.

Role Variables
--------------

Setting the `cleanmgr_install_reboot` value to `true` allows Ansible to reboot
the target Windows computer after installing the Desktop Experience feature.
Ansible doesn't get a result from `cleanmgr.exe` to know if it should reboot or not, so toggling `cleanmgr_cleanmgr_reboot` will always or never reboot after running `cleanmgr.exe`.

    cleanmgr_install_reboot: true
    cleanmgr_cleanmgr_reboot: false
    cleanmgr_sagerun: "0001"

    cleanmgr_active_setup_temp_folders: false
    cleanmgr_branchcache: false
    cleanmgr_downloaded_program_files: true
    cleanmgr_install_path: C:/Windows/System32/cleanmgr.exe
    cleanmgr_internet_cache_files: true
    cleanmgr_memory_dump_files: false
    cleanmgr_old_chkdsk_files: false
    cleanmgr_previous_installations: false
    cleanmgr_recycle_bin: false
    cleanmgr_service_pack_cleanup: false
    cleanmgr_setup_log_files: false
    cleanmgr_system_error_memory_dump_files: false
    cleanmgr_system_error_minidump_files: false
    cleanmgr_temporary_files: false
    cleanmgr_temporary_setup_files: false
    cleanmgr_thumbnail_cache: false
    cleanmgr_upgrade_discarded_files: false
    cleanmgr_user_file_versions: false
    cleanmgr_windows_defender: false
    cleanmgr_windows_error_reporting_archive_files: false
    cleanmgr_windows_error_reporting_queue_files: false
    cleanmgr_windows_error_reporting_system_archive_files: false
    cleanmgr_windows_error_reporting_system_queue_files: false
    cleanmgr_windows_esd_installation_files: false
    cleanmgr_update_cleanup: true
    cleanmgr_windows_upgrade_log_files: false

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: deekayen.cleanmgr, cleanmgr_install_reboot: false }

Requirements
------------

Windows 2012R2+. Executing cleanmgr.exe by Powershell over winrm on Windows 2008 doesn't seem to work well.

Dependencies
------------

None.

License
-------

BSD
