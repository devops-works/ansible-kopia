# Kopia ansible role

Ansible role for https://kopia.io/

Installs kopia and connects the root user to a repository, on GCS or on S3
(AWS or any S3-compatible storage). The repository is created when it does not
exist yet. Scheduling snapshots is left to the caller.

## Role Variables

### GCS

Configured when `kopia_gcs_bucket` is defined.

- `kopia_gcs_bucket`: GCS destination bucket for Kopia
- `kopia_gcs_email`: email for SA
- `kopia_gcs_password`: repos password

### S3

Configured when `kopia_s3_bucket` is defined.

| variable | default | purpose |
|---|---|---|
| `kopia_s3_bucket` | none | bucket name |
| `kopia_s3_access_key` | none | access key |
| `kopia_s3_secret_key` | none | secret key |
| `kopia_s3_password` | none | repository password |
| `kopia_s3_endpoint` | `s3.amazonaws.com` | endpoint host, without scheme |
| `kopia_s3_region` | empty | region, when the storage needs one |
| `kopia_s3_prefix` | empty | prefix in the bucket; end it with `/` for a directory |
| `kopia_s3_create` | `true` | create the repository when connecting fails |
| `kopia_s3_readonly` | `false` | connect read-only; never creates nor changes repository settings |
| `kopia_s3_retention_mode` | empty | object lock mode, `GOVERNANCE` or `COMPLIANCE` |
| `kopia_s3_retention_period` | empty | object lock duration, Go duration (`2160h`) |
| `kopia_s3_manage_maintenance` | `true` | set the maintenance owner and parameters below |
| `kopia_s3_full_maintenance_interval` | `24h` | interval of full maintenance |
| `kopia_s3_extend_object_locks` | `false` | let full maintenance extend the locks of blobs still in use |
| `kopia_policy_global_flags` | `[]` | arguments for `kopia policy set --global` |

Credentials and the repository password are passed to kopia through the
environment, never on the command line.

#### Object lock

The bucket must be created with object lock enabled, before the role runs. The
retention mode and period are stored in the repository when it is created;
setting the variables later does not change an existing repository (use
`kopia repository set-parameters` for that).

Kopia never deletes an object version, it only adds delete markers, so the key
needs no version-delete permission. Space is reclaimed by a lifecycle rule on
the bucket that expires noncurrent versions.

With `kopia_s3_extend_object_locks`, full maintenance must actually run, at
least once per retention period, on the host that owns maintenance (the role
makes the current host the owner unless `kopia_s3_readonly` is set).

## Example Playbook

```yaml
  - hosts: servers
    roles:
        - { role: ansible-kopia, kopia_gcs_bucket: "foo-1234" }
```

```yaml
  - hosts: servers
    roles:
      - role: ansible-kopia
        kopia_s3_bucket: backups
        kopia_s3_endpoint: sos-ch-dk-2.exo.io
        kopia_s3_region: ch-dk-2
        kopia_s3_access_key: "{{ vault_kopia_access_key }}"
        kopia_s3_secret_key: "{{ vault_kopia_secret_key }}"
        kopia_s3_password: "{{ vault_kopia_password }}"
        kopia_s3_retention_mode: COMPLIANCE
        kopia_s3_retention_period: 2160h
        kopia_s3_extend_object_locks: true
```

## License

MIT
