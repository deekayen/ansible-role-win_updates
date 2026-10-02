# deekayen.win_updates

[![CI](https://github.com/deekayen/ansible-role-win_updates/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-win_updates/actions/workflows/ci.yml) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-deekayen.win__updates-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/deekayen/win_updates/) [![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue) ![Windows platform](https://img.shields.io/badge/platform-windows-lightgrey)

An Ansible role that installs Windows updates from the categories you choose, reboots the host when the updates ask for it, and prints a summary of what was installed.

The role wraps the [`ansible.windows.win_updates`](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_updates_module.html) module with `category_names` set from `win_updates_category_names`. It then calls `ansible.windows.win_reboot` with 3600-second shutdown and reboot timeouts when the result has `reboot_required`, and finishes with a debug message listing the installed update titles, the failed update count, and the reboot time in minutes.

## Requirements

- ansible-core 2.15 or newer on the controller.
- The `ansible.windows` collection: `ansible-galaxy collection install ansible.windows`.
- A WinRM or SSH connection to the target with administrative rights.
- Network access from the target to Windows Update or to the update server its policy points at.

## Supported platforms

`meta/main.yml` declares Windows, all versions. CI lints the role and runs `ansible-playbook --syntax-check`; it does not apply the role to a Windows host.

## Installation

From Ansible Galaxy:

```bash
ansible-galaxy role install deekayen.win_updates
ansible-galaxy collection install ansible.windows
```

Or pin it in `requirements.yml`:

```yaml
---
roles:
  - name: deekayen.win_updates
    src: https://github.com/deekayen/ansible-role-win_updates.git
    scm: git
    version: main

collections:
  - name: ansible.windows
```

```bash
ansible-galaxy install -r requirements.yml
```

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `win_updates_category_names` | `['CriticalUpdates', 'SecurityUpdates', 'UpdateRollups']` | Update categories to install. Must be a non-empty list drawn from `*`, `Application`, `Connectors`, `CriticalUpdates`, `DefinitionUpdates`, `DeveloperKits`, `FeaturePacks`, `Guidance`, `SecurityUpdates`, `ServicePacks`, `Tools`, `UpdateRollups`, `Updates`, and `Upgrades`; `tasks/assert.yml` enforces this. |
| `win_updates_reboot` | `false` | When the first install fails, reboot and run the install a second time instead of failing. See [Known issues](#known-issues). |

## Behavior

- **The role reboots the host whenever the final `win_updates` run reports `reboot_required`, whatever `win_updates_reboot` is set to.** `win_updates_reboot` only controls the retry after a failed first install. Run the role in a maintenance window.
- With `win_updates_reboot: true`, any failure of the first install starts the retry, not only a pending-reboot failure.
- The summary task runs on every play, including when no updates were installed.

## Dependencies

None. The `ansible.windows` collection is a requirement, not a role dependency.

## Example playbook

```yaml
---
- name: Install critical and security updates.
  hosts: windows_patch_group_a

  vars:
    win_updates_category_names:
      - CriticalUpdates
      - SecurityUpdates
    win_updates_reboot: true

  roles:
    - deekayen.win_updates
```

## Known issues

- The retry block's reboot, at `tasks/main.yml:28-30`, runs only when `win_updates_result.error` equals `"A reboot is required before more updates can be installed."`. Neither `ansible.windows` release checked (2.7.0 and 3.8.0) returns an `error` key. Release 3.8.0 reports the failure in `msg`, and the message both releases throw has no trailing period. With `win_updates_reboot: true`, the role therefore skips that reboot and runs `win_updates` again at once, which fails the same way when a reboot is pending.

## Development

CI runs on every push to `main` and every pull request (see `.github/workflows/ci.yml`). It installs `ansible.windows`, runs `ansible-lint --profile production`, and syntax-checks `tests/test.yml`. To run the same checks locally:

```bash
pip3 install ansible-lint
ansible-galaxy collection install ansible.windows
ansible-lint --profile production
mkdir -p .ansible/roles && ln -sfn "$PWD" .ansible/roles/deekayen.win_updates
ANSIBLE_ROLES_PATH=.ansible/roles:~/.ansible/roles ansible-playbook --syntax-check tests/test.yml -i tests/inventory
```

The repository also has a `.pre-commit-config.yaml`; run `pre-commit run --all-files` before pushing.

### Repository layout

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Update install, retry block, reboot, and summary. |
| `tasks/assert.yml` | Checks `win_updates_category_names`, tagged `always`. |
| `defaults/main.yml` | Every user-facing variable. |
| `meta/argument_specs.yml` | Argument types and descriptions. |
| `tests/` | Syntax-check playbook and inventory used by CI. |

## Releases

Pushing a git tag runs `.github/workflows/release.yml`, which imports the tagged commit into Ansible Galaxy as `deekayen.win_updates`. The import needs a `GALAXY_API_KEY` repository or organization secret.

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Author

[David Norman](https://github.com/deekayen). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
