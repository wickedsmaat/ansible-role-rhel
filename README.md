# ansible-role-rhel
Overview

This role establishes a baseline configuration for RHEL-based homelab servers.
It manages SSH hardening, system users, sudo access, subscription registration, repository configuration, package installation, SELinux/firewall settings, networking defaults, and system updates.

It is meant to be the first role run on any new server, creating a consistent and predictable operating environment for all subsequent roles.

## Task-by-Task Breakdown
Deploying SSH Configuration Snippets

This task installs multiple drop-in configuration files under /etc/ssh/sshd_config.d/.
These files may enforce key-only authentication, disable root login, or restrict algorithms.
Centralizing SSH rules creates security consistency across all homelab servers.

Deploying the Main sshd_config

The role overwrites /etc/ssh/sshd_config with a version that references the drop-in directory.
This ensures custom SSH policies take effect before the service starts.
It allows full control of SSH hardening through versioned config templates.

Restarting the SSH Service

This ensures all new SSH rules take immediate effect.
Restarting the service validates syntax and applies new security settings.
This is essential to enforce consistent access patterns across the environment.

Ensuring /root/.ssh Exists

This creates the .ssh directory for the root account.
This is required for storing administrator public keys.
Without it, key-based login for root would fail.

Ensuring /etc/skel/.ssh Exists

This task ensures that any newly created user receives an .ssh directory automatically.
This provides a standard environment for adding authorized keys.
It keeps user home directory layouts consistent across systems.

Installing Shell Dotfiles for root

This task deploys .bashrc, .bash_profile, .bash_logout, and .vimrc into /root.
These files enhance the interactive shell experience and define consistent defaults.
This provides predictable behavior when performing admin tasks.

Installing Dotfiles into /etc/skel

The same dotfiles are copied into /etc/skel, so newly created users inherit the same environment.
This enforces identical shell behavior for all homelab user accounts.
It prevents drift between servers and between accounts.

Deploying Authorized Keys to root

The role places pre-approved SSH public keys into /root/.ssh/authorized_keys.
This enables immediate key-based login after system provisioning.
It eliminates the insecure and unnecessary step of enabling password authentication.

Deploying Authorized Keys into /etc/skel

Any new user created later will automatically inherit authorized keys.
This maintains secure login defaults across future accounts.
It ensures new users do not accidentally fall back to password login.

Creating the Admin User

The misteralexander user is created with a hashed password and home directory.
This provides a non-root administrative account for daily use.
Using a standard user with sudo is a best practice for security and auditing.

Creating Sudoers Entries for Admin and Automation Users

This task places sudoers files under /etc/sudoers.d/.
It defines passwordless sudo for both the admin account and the automation account.
This ensures Ansible and the primary operator can perform privileged actions without friction.

Ensuring Root Account Has a Password and Unlocked Access

This task assigns a known password to root and ensures the account is not locked.
This enables emergency console access when SSH is unavailable.
It is a safety measure for homelab environments where console access is common.

Registering the System with Subscription Manager

This registers the machine using an activation key and organization ID.
Registration enables all RHEL official repositories and updates.
Without this, the system cannot install or update packages normally.

Clearing All RHSM Repository Assignments

This disables all RHSM-provided repositories.
It creates a clean baseline to selectively re-enable only needed repos.
This avoids accidental installation of unneeded or unsupported packages.

Enabling Required RHEL Repositories

This role enables BaseOS, AppStream, CRB, HA, plus any required extension repositories.
These provide core system packages, libraries, developer tools, and optional features.
This selective enablement prevents repository bloat and improves reliability.

Importing GPG Keys for EPEL and RPMFusion

These tasks install cryptographic signing keys required for verifying external repositories.
Without these keys, package installation from EPEL or RPMFusion would fail.
This step secures the software supply chain.

Adding EPEL, RPMFusion Free, and RPMFusion Non-Free Repositories

These repositories provide community-maintained packages not shipped by RHEL.
They unlock access to media codecs, utilities, monitoring tools, and more.
These repositories greatly expand software availability for homelab use.

Installing Core System Packages

This large task installs:

Developer tools

Monitoring utilities

Storage tools

CLI enhancements

Networking tools

Python development packages

This transforms a minimal install into a fully functional homelab server environment.

Upgrading PIP

This updates Python’s package installer to the newest version.
A modern pip ensures compatibility with current Python packages.
This matters for any automation or scripting roles using Python.

Applying All System Updates

This updates all installed packages to their latest patch level.
It ensures the server is secure and up to date immediately after provisioning.
If a new kernel is installed, this triggers a reboot.

Rebooting if Required

If updates include kernel changes or SELinux transitions, the system reboots.
This ensures all configuration changes take effect properly.
This produces a clean, stable post-provisioning state.

## Summary

This baseline role prepares a RHEL server for long-term homelab use.
It configures SSH, users, repositories, packages, security defaults, and system updates — ensuring that every machine in the environment is reliable, secure, and consistent.