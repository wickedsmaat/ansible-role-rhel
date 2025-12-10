# ansible-role-rhel

## Overview

This role configures a RHEL-based system according to your homelab’s baseline standards.
It performs Red Hat subscription registration, applies system purpose metadata, manages default user accounts, deploys authorized SSH keys, enforces package updates, and reboots the machine when required.

The result is a consistently registered, patched, and access-controlled Red Hat Enterprise Linux environment suitable for both homelab and production-tier automation.

---

## Required Variables

This role **cannot run** unless the following variables are defined.
They represent subscription information, default account policies, SSH access configuration, and system purpose details.

### Red Hat Subscription Variables

| Variable                                           | Description                                              |
| -------------------------------------------------- | -------------------------------------------------------- |
| `rhsm_sub_info.state`                              | Whether the subscription should be `present` or `absent` |
| `rhsm_sub_info.activationkey`                      | RHSM activation key used for registration                |
| `rhsm_sub_info.org_id`                             | Organization ID associated with the subscription         |
| `rhsm_sub_info.syspurpose.usage`                   | System Purpose usage classification                      |
| `rhsm_sub_info.syspurpose.role`                    | System Purpose role classification                       |
| `rhsm_sub_info.syspurpose.service_level_agreement` | SLA level for entitlement tracking                       |
| `rhsm_sub_info.syspurpose.sync`                    | Whether to sync system purpose (`true` recommended)      |
| `rhsm_sub_info.force_register`                     | Force a re-registration if necessary (`true`)            |

### Default User Variables

Each must contain the following keys:

| Variable              | Required Keys                                                          |
| --------------------- | ---------------------------------------------------------------------- |
| `default_redhat_user` | `name`, `state`, `create_home`, `shell`, `password`, `update_password` |
| `default_redhat_root` | `name`, `state`, `create_home`, `shell`, `password`, `update_password` |

### SSH Authorization Variables

| Variable              | Description                                                |
| --------------------- | ---------------------------------------------------------- |
| `ssh_authorized_keys` | A **list** of authorized SSH keys, each containing `value` |

Example structure:

```yaml
ssh_authorized_keys:
  - value: "ssh-ed25519 AAAA... user@example"
```

---

## Where to Define These Variables

You may define these values:

* in the **calling playbook**
* in **group_vars** or **host_vars**
* or passed inline when including the role

---

## Example Playbook

```yaml
- name: Apply Red Hat baseline configuration
  hosts: all

  vars:
    rhsm_sub_info:
      state: present
      activationkey: "HOMELAB-KEY"
      org_id: "1234567"
      syspurpose:
        usage: "Production"
        role: "Server"
        service_level_agreement: "Standard"
        sync: true
      force_register: true

    default_redhat_user:
      name: "alex"
      state: present
      create_home: true
      shell: "/bin/bash"
      password: "{{ 'mypassword' | password_hash('sha512') }}"
      update_password: "on_create"

    default_redhat_root:
      name: "root"
      state: present
      create_home: false
      shell: "/bin/bash"
      password: "{{ 'rootpassword' | password_hash('sha512') }}"
      update_password: "on_create"

    ssh_authorized_keys:
      - value: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... alex@snyderfamily"

  roles:
    - redhat_baseline
```

---

## Task-by-Task Breakdown

### Validating Required Variables

The role begins by validating that all subscription, user, and SSH key variables are present and correctly structured.
This prevents partial registration, broken user accounts, or incomplete system metadata.

### Registering the Host with Red Hat Subscription Management

The role configures RHSM using the provided:

* activation key
* organization ID
* system purpose metadata

It also supports forced re-registration when a system’s identity must be refreshed.

### Creating the Default Red Hat User

The role enforces a consistent default user configuration, including:

* username
* password policy
* shell
* home directory creation
* password update behavior

This user becomes the primary non-root login identity.

### Setting Root Account Attributes

The same enforcement is applied to the root account:

* shell
* password behavior
* update policy

This ensures the host complies with security and automation requirements.

### Deploying Authorized SSH Keys

Each key listed in `ssh_authorized_keys` is installed into the appropriate user’s account.
This ensures passwordless access for automation, administrative login, or homelab workflows.

### Applying System Updates

The role updates all system packages using:

```yaml
ansible.builtin.dnf:
  name: '*'
  state: latest
```

This ensures the host is fully patched before continuing.

### Rebooting When Updates Require It

If updates changed system libraries or kernel components, a reboot is triggered automatically with a controlled timeout.

This guarantees the system boots with the latest security patches and system purpose metadata.

---

## Summary

This role automates the complete Red Hat baseline configuration process.
It registers the system, sets system purpose metadata, provisions required user accounts, deploys SSH keys, performs updates, and reboots when necessary.

By defining the **Required Variables**, you ensure that each RHEL host is consistently configured, properly registered, and fully ready for downstream automation or homelab workloads.
