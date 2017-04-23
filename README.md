# Ansible Role: Unattended Upgrades

An ansible role to set up unattended upgrades on Ubuntu. Only security updates are enabled by default.

## Requirements

> This role has been tested on `Ubuntu 16.04` and `Ubuntu 16.10` only.

For Debian, you might need to change the `unattended_origins_patterns` value as the default targes Ubuntu.

### Unattended Mail

If you set `unattended_mail` to an e-mail address, make sure `mailx` command is available and your system is able to send e-mails.

### Automatic Reboot

If you enable automatic reboot feature (`unattended_automatic_reboot`), the role will attempt to install `update-notifier-common` package, which is required on some systems for detecting and executing reboot after the upgrade. You may optionally define a specific time for rebooting (`unattended_automatic_reboot_time`).

## Variables

- `unattended_origins_patterns`: array of origins patterns to determine whether the package can be automatically installed, for more details see [Origins Patterns](#origins-patterns) below.
  - Default: `['origin=Ubuntu,archive=${distro_codename}-security']`
- `unattended_package_blacklist`: packages which won't be automatically upgraded
  - Default: `[]`
- `unattended_autofix_interrupted_dpkg`: whether on unclean dpkg exit to run `dpkg --force-confold --configure -a`
  - Default: `true`
- `unattended_minimal_steps`: split the upgrade into the smallest possible chunks so that they can be interrupted with SIGUSR1.
  - Default: `false`
- `unattended_install_on_shutdown`: install all unattended-upgrades when the machine is shuting down.
  - Default: `false`
- `unattended_mail`: e-mail address to send information about upgrades or problems with unattended upgrades
  - Default: `false` (don't send any e-mail)
- `unattended_mail_only_on_error`: send e-mail only on errors, otherwise e-mail will be sent every time there's a package upgrade.
  - Default: `false`
- `unattended_remove_unused_dependencies`: do automatic removal of new unused dependencies after the upgrade.
  - Default: `false`
- `unattended_automatic_reboot`: Automatically reboot system if any upgraded package requires it, immediately after the upgrade.
  - Default: `false`
- `unattended_automatic_reboot_time`: Automatically reboot system if any upgraded package requires it, at the specific time (_HH:MM_) instead of immediately after the upgrade.
  - Default: `false`
- `unattended_ignore_apps_require_restart`: unattended-upgrades won't automatically upgrade some critical packages requiring restart after an upgrade (i.e. there is `XB-Upgrade-Requires: app-restart` directive in their debian/control file). With this option set to `true`, unattended-upgrades will upgrade these packages regardless of the directive.
  - Default: `false`

## Origins Patterns

Origins Pattern is a more powerful alternative to the Allowed Origins option used in previous versions of unattended-upgrade.

Pattern is composed from specific keywords:

- `a`,`archive`,`suite` – e.g. `stable`, `trusty-security` (`archive=stable`)
- `c`,`component`   – e.g. `main`, `crontrib`, `non-free` (`component=main`)
- `l`,`label` – e.g. `Debian`, `Debian-Security`, `Ubuntu`
- `o`,`origin` – e.g. `Debian`, `Unofficial Multimedia Packages`, `Ubuntu`
- `n`,`codename` – e.g. `jessie`, `jessie-updates`, `trusty` (this is only supported with `unattended-upgrades` >= 0.80)
- `site` – e.g. `http.debian.net`

You can review the available repositories using `apt-cache policy` and debug your choice using `unattended-upgrades -d` command on a target system.

Additionally unattended-upgrades support two macros (variables), derived from `/etc/debian_version`:

- `${distro_id}` – Installed distribution name, e.g. `Debian` or `Ubuntu`.
- `${distro_codename}` – Installed codename, e.g. `jessie` or `trusty`.

Using `${distro_codename}` should be preferred over using `stable` or `oldstable` as a selected, as once `stable` moves to `oldstable`, no security updates will be installed at all, or worse, package from a newer distro release will be installed by accident. The same goes for upgrading your installation from `oldstable` to `stable`, if you forget to change this in your origin patterns, you may not receive the security updates for your newer distro release. With `${distro_codename}`, both cases can never happen.

## Usage Example

```yaml
- hosts: all
  vars:
    unattended_origins_patterns:
      - 'origin=Ubuntu,archive=${distro_codename}-security'
      - 'o=Ubuntu,a=${distro_codename}-updates'
    unattended_package_blacklist: [cowsay, vim]
    unattended_mail: 'root@example.com'
  roles:
    - setup_unattended_upgrades
```


### Patterns Examples

By default, only security updates are allowed. You can add more patterns to allow unattended-updates install more packages automatically, however be aware that automated major updates may potentially break your system.

In Ubuntu, archive always contains the distribution codename

```yaml
unattended_origins_patterns:
  - 'origin=Ubuntu,archive=${distro_codename}-security'
  - 'o=Ubuntu,a=${distro_codename}'
  - 'o=Ubuntu,a=${distro_codename}-updates'
  - 'o=Ubuntu,a=${distro_codename}-proposed-updates'
```


## License

MIT / BSD

## Author Information

This role was created by [thedumbtechguy](https://github.com/thedumbtechguy) | [twitter](https://github.com/thedumbtechguy) | [blog](https://thedumbtechguy.blogspot.com).

## Credits

This role was built upon the original work of:

- [jnv/ansible-role-unattended-upgrades](https://github.com/jnv/ansible-role-unattended-upgrades)