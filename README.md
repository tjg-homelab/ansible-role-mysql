# Ansible Role: mysql

[![CI](https://github.com/tjg-homelab/ansible-role-mysql/actions/workflows/ci.yml/badge.svg)](https://github.com/tjg-homelab/ansible-role-mysql/actions/workflows/ci.yml)

Installs and configures MySQL (or MariaDB) on Debian and RedHat family
systems. Sets the root password, performs secure-installation cleanup
(removes anonymous users, hostname root accounts, and the test database),
writes a root `~/.my.cnf`, and optionally creates a remote-access account and
opens the server to the network.

## Security posture

**This role is secure by default:** MySQL binds to `127.0.0.1` and no remote
account is created. Network exposure is opt-in — set
`mysql_enable_remote_network_access: true` and provide remote credentials, and
only do so behind a trusted network boundary. This is the opposite of a stock
MySQL "listen everywhere" configuration by design.

## Requirements

- Debian 12/13, Ubuntu 22.04/24.04, or Enterprise Linux 9
- The `community.mysql` collection (`ansible-galaxy collection install community.mysql`)
- `mysql_root_password` must be supplied — store it in Ansible Vault

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `mysql_root_password` | _(required)_ | Password for `root@localhost` |
| `mysql_manage_secure_installation` | `true` | Remove anonymous/hostname/test artifacts |
| `mysql_enable_remote_network_access` | `false` | Open the server to the network |
| `mysql_bind_address` | `127.0.0.1` | Listen address (set `0.0.0.0` for remote) |
| `mysqlx_bind_address` | `127.0.0.1` | X protocol listen address (`""` to skip) |
| `mysql_port` | `3306` | Listen port |
| `mysql_manage_remote_user` | `true` | Create the remote account (when credentials given) |
| `mysql_remote_username` | _(unset)_ | Remote account name |
| `mysql_remote_pass` | _(unset)_ | Remote account password (vault) |
| `mysql_remote_user_host` | `"%"` | Host pattern the remote account may connect from |
| `mysql_remote_user_priv` | `"*.*:ALL,GRANT"` | Privileges for the remote account |

OS-specific package/path variables (`mysql_server_package_debian`,
`mysql_service_name_debian`, `mysql_config_file_debian`, etc.) are overridable
— see `defaults/main.yml`. Overriding them to MariaDB values lets the role
manage MariaDB as well.

## Example Playbook

Local-only database (default posture):

```yaml
- hosts: db
  roles:
    - role: mysql
      vars:
        mysql_root_password: "{{ vault_mysql_root_password }}"
```

Network-accessible with a remote app account:

```yaml
- hosts: db
  roles:
    - role: mysql
      vars:
        mysql_root_password: "{{ vault_mysql_root_password }}"
        mysql_enable_remote_network_access: true
        mysql_bind_address: "0.0.0.0"
        mysql_remote_username: appuser
        mysql_remote_pass: "{{ vault_mysql_app_password }}"
        mysql_remote_user_priv: "appdb.*:ALL"
```

Installing via `requirements.yml`:

```yaml
roles:
  - name: mysql
    src: https://github.com/tjg-homelab/ansible-role-mysql.git
    version: v1.0.0
```

## Testing

Molecule (Docker driver) converges, checks idempotence, and verifies against
Debian 12 (MariaDB) and Ubuntu 24.04 (MySQL). The scenario enables remote
access to exercise the bind-address and remote-user paths, then asserts the
root client config, running service, bind configuration, and remote account.

```bash
pip install ansible-core molecule molecule-plugins[docker] docker
ansible-galaxy collection install community.docker community.mysql ansible.posix
molecule test
```

## License

MIT

## Author

Rodney Nissen ([The Jira Guy](https://thejiraguy.com))
