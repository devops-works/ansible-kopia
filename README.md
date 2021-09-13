# Kopia ansible role

Ansible role for https://kopia.io/

Only supports GCS.

## Role Variables

- `kopia_gcs_bucket`: GCS destination bucket for Kopia

## Example Playbook

```yaml
  - hosts: servers
    roles:
        - { role: ansible-kopia, kopia_gcs_bucket: "foo-1234" }
```

## License

MIT
