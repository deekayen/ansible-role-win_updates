Windows Updates
===============

Install Microsoft Windows updates.

Requirements
------------

The target Windows machines must have whitelisted network access to download updates.

Role Variables
--------------

By default, this role installs the default categories of updates provided by the Ansible [win_updates](http://docs.ansible.com/ansible/win_updates_module.html) module. Change `win_updates_category_names` to install alternate categories of updates.

Not all updates request to reboot after installation, but they do tell Ansible when a reboot is required to complete the install. Toggle the `win_updates_reboot` variable to `true` to reboot when updates request a reboot.

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: servers
      roles:
         - role: deekayen.win_updates
           win_updates_reboot: true
           win_updates_category_names:
             - "CriticalUpdates"
             - "SecurityUpdates"

License
-------

BSD
