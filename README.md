Windows Updates
===============

Install Windows updates.

Requirements
------------

The target Windows machines must have whitelisted network access to download updates.

Role Variables
--------------

By default, this role installs the default categories of updates provided by the Ansible [win_updates](http://docs.ansible.com/ansible/win_updates_module.html) module. Change `win_updates_category_names` to install alternate categories of updates.

To reboot Windows after successful installation, toggle the `win_updates_reboot` variable to `true`.

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: deekayen.win_updates, win_updates_reboot: true }

License
-------

BSD
