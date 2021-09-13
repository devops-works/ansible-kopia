# Kopia ansible role

Ansible role for https://kopia.io/

Only supports GCS.

## Role Variables

- `kopia_gcs_bucket`: GCS destination bucket for Kopia
- `kopia_gcs_email`: email for SA
- `kopia_gcs_password`: repos password


## Example Playbook

```yaml
  - hosts: servers
    roles:
        - { role: ansible-kopia, kopia_gcs_bucket: "foo-1234" }
```

## License

MIT
