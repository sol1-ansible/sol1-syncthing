# Ansible Role: sol1-syncthing

Installs and configures [Syncthing](https://syncthing.net/) for continuous file synchronization between hosts with automatic device discovery.

## Requirements

- Debian/Ubuntu based operating system
- Ansible 2.10+
- `community.general` collection (for XML parsing)
- `acl` package on target hosts (for folder permissions when syncing non-owned directories)

## Role Variables

### Required Variables

These variables must be defined when using the role:

```yaml
syncthing_gui_user: "admin"           # Username for Syncthing Web GUI
syncthing_gui_password: "password"    # Password for Syncthing Web GUI
```

### Device Configuration

The role supports **automatic device discovery** from all hosts in the current play. Devices are auto-discovered and paired unless you manually define `syncthing_devices`:

```yaml
# Auto-discovery (default) - leave empty or omit
syncthing_devices: []

# Manual device configuration (optional)
syncthing_devices:
  - id: "XXXXXXX-XXXXXXX-XXXXXXX-XXXXXXX-XXXXXXX-XXXXXXX-XXXXXXX-XXXXXXX"
    name: "remote-host"
    auto_accept: true
```

### Folder Configuration

Define folders to sync with flexible ownership and sync types:

```yaml
syncthing_folders:
  - id: "backups"
    label: "System Backups"
    path: "/var/backups"
    owner: "backup"
    group: "backup"
    mode: "0755"
    create: false               # Use existing directory
    type: "sendonly"
    rescan_interval: 300
    devices:                    # Optional: specific devices, or omit to share with all
      - "DEVICE_ID_1"
  
  - id: "shared_data"
    label: "Shared Data"
    type: "receiveonly"
    # path defaults to: {{ syncthing_storage_root_dir }}/{{ id }}
    # owner/group default to syncthing_user/syncthing_group
    # create defaults to true
```

### Optional Variables

These variables have sensible defaults but can be overridden:

```yaml
# Storage configuration
syncthing_storage_root_dir: "/srv/syncthing"

# Service user configuration
syncthing_user: "syncthing"
syncthing_group: "syncthing"
syncthing_user_uid: 9001

# Configuration paths
syncthing_config_dir: "/home/{{ syncthing_user }}/.config/syncthing"
syncthing_config_file: "{{ syncthing_config_dir }}/config.xml"

# API settings
syncthing_api_url: "https://localhost:8384"
syncthing_api_verify_ssl: false

# Service settings
syncthing_service_name: "syncthing"
syncthing_service_enabled: true
syncthing_service_state: "started"
```

## Dependencies

- `community.general` collection

Install with:

```bash
ansible-galaxy collection install community.general
```

## Example Playbook

### Automatic Device Discovery (Single Run)

The role automatically discovers all hosts in the play and pairs them:

```yaml
- hosts: app-server-1,app-server-2,app-server-3
  roles:
    - role: sol1-syncthing
      vars:
        syncthing_gui_user: "admin"
        syncthing_gui_password: "{{ vault_syncthing_password }}"
        syncthing_folders:
          - id: "app_data"
            label: "Application Data"
            path: "/opt/app/data"
            type: "sendonly"
```

All three hosts will automatically discover and pair with each other.

### Multiple Folders with Different Owners

```yaml
- hosts: backup_servers
  roles:
    - role: sol1-syncthing
      vars:
        syncthing_gui_user: "admin"
        syncthing_gui_password: "{{ vault_syncthing_password }}"
        syncthing_folders:
          - id: "system_backups"
            path: "/var/backups"
            owner: "backup"
            group: "backup"
            create: false
            type: "sendonly"
          
          - id: "mysql_backups"
            path: "/var/lib/mysql/backups"
            owner: "mysql"
            group: "mysql"
            create: false
            type: "sendonly"
          
          - id: "logs"
            path: "/var/log/app"
            owner: "syslog"
            group: "adm"
            mode: "0750"
            create: false
            type: "sendonly"
```

### Manual Device Configuration

If you prefer manual control over device pairing:

```yaml
- hosts: syncthing_servers
  roles:
    - role: sol1-syncthing
      vars:
        syncthing_gui_user: "admin"
        syncthing_gui_password: "{{ vault_syncthing_password }}"
        syncthing_devices:
          - id: "ABCDEF-123456-..."
            name: "server-1"
            auto_accept: true
          - id: "GHIJKL-789012-..."
            name: "server-2"
            auto_accept: true
        syncthing_folders:
          - id: "shared_data"
            type: "sendonly"
```

## License

MIT

## Author Information

This role was created by [Sol1](https://sol1.com.au/).
