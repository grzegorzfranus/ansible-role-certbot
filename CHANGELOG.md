# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.3] - 2026-05-10

### Fixed 🔧
- Fixed Molecule CI failure on Debian 12 Docker images caused by pip-installed `cryptography`/`pyopenssl` shadowing compatible apt system packages
- Remove pip-installed crypto packages in Molecule `prepare.yml` to let system apt versions work with `certbot` 2.1.0

## [1.0.2] - 2026-05-10

### Added ✅
- `certbot_purge_unmanaged` variable — when `true`, deletes certificates not defined in `certbot_domains`
- `tasks/purge.yml` — lists server certificates, diffs against managed list, removes unmanaged
- Purge plan output at verbosity 0 (always visible for destructive operations)
- Example 7 in README: "Enforce Certificate Inventory" with purge enabled

### Changed 🔄
- Updated README.md with purge variable, tag, file structure, and example
- Updated `meta/argument_specs.yml` with `certbot_purge_unmanaged` definition
- Renumbered `defaults/main.yml` sections (revocation is now §12)

## [1.0.1] - 2026-05-06

### Fixed 🔧
- Fixed environment validation assertion checking `certbot_environment` against `__certbot_supported_validation_methods` instead of `['staging', 'production']`

### Added ✅
- `.ansible-lint` configuration (shared profile, role-name skip)
- `.gitignore` for IDE, cache, and Python environment files
- `.yamllint` configuration (200 char line limit, truthy disabled, octal-values forbidden)
- GitHub Actions CI workflow (`test-and-validation.yml`) with yamllint + Molecule matrix
- GitHub Actions Galaxy publish workflow (`publish-to-galaxy.yml`)
- Molecule test scenario `default` — validates package installation, directories, credential file permissions
- Molecule test scenario `renewal` — validates systemd timer, cron deployment, and cross-cleanup
- Molecule test scenario `uninstall` — validates cleanup via `certbot_state: absent`

### Changed 🔄
- Removed unused `files/` directory
- Updated README.md with CI/CD, Testing, and corrected file structure sections

## [1.0.0] - 2026-05-04

### Added ✅
- 🚀 **Initial release** of Ansible role for Certbot / Let's Encrypt certificate management
- 🌐 **Multi-platform support** for Debian 12 (Bookworm) and Ubuntu 24.04 (Noble)
- 🔐 **HTTP validation methods**: standalone, webroot, nginx, and apache authenticators
- 🌍 **DNS validation providers**: Cloudflare, Route53 (AWS), and DigitalOcean
- 🃏 **Wildcard certificate support** via DNS-01 challenge validation
- 🔄 **Staging and production** ACME environment selection
- ⏰ **Auto-renewal** via systemd timer or cron with configurable schedules
- 🪝 **Renewal hooks** support (pre, post, deploy) for service restarts and custom actions
- 🧪 **Dry-run renewal testing** for validation without modifying certificates
- ❌ **Certificate revocation** with reason codes and optional post-revoke cleanup
- 🔑 **ECDSA and RSA** key type support with configurable parameters
- 📋 **`meta/argument_specs.yml`** for Ansible-native argument validation (CoP §3.1.20)
- ✅ **Comprehensive variable assertion** via `tasks/assert.yml` with runtime validation
- 🛡️ **Secure credential management** with `no_log`, `0600` permissions, and Vault integration
- 📦 **Automatic plugin installation** for DNS and web server authenticators

### Core Features 🎯
- **Certificate Issuance**: Automated certificate requests with per-domain validation method override
- **Multi-Domain Support**: Multiple SANs per certificate with flexible domain configuration
- **DNS Providers**: Cloudflare (API token), Route53 (access key), DigitalOcean (API token)
- **Web Server Integration**: Nginx and Apache plugins for zero-downtime certificate deployment
- **Renewal Management**: Systemd timer or cron-based automatic renewal with hook support
- **Revocation**: Certificate revocation with RFC 5280 reason codes and optional file cleanup
- **Environment Control**: Staging environment for testing without rate limits

### Technical Specifications 🛠️
- **Minimum Ansible Version**: 2.16+
- **Python Version**: 3.9+
- **License**: Apache-2.0
- **Company**: EWARE
- **Author**: Grzegorz Franus
- **Quality**: All yamllint and ansible-lint checks passing

### Configuration Variables 📝
- Installation: `certbot_state`, `certbot_upgrade`
- Account: `certbot_email`, `certbot_agree_tos`, `certbot_environment`
- Validation: `certbot_validation_method`, `certbot_http_method`, `certbot_dns_provider`
- Domains: `certbot_domains` (list of certificate definitions)
- DNS Credentials: `certbot_cloudflare_api_token`, `certbot_route53_access_key_id`, `certbot_digitalocean_token`
- Key Settings: `certbot_key_type`, `certbot_rsa_key_size`, `certbot_elliptic_curve`
- Renewal: `certbot_auto_renewal_enabled`, `certbot_renewal_method`, `certbot_renewal_schedule`
- Revocation: `certbot_revoke_certs`, `certbot_revoke_reason`, `certbot_revoke_delete_after`

### File Structure 📁
```
ansible-role-certbot/
├── defaults/main.yml           # Default configuration variables
├── handlers/main.yml           # Service restart and reload handlers
├── meta/
│   ├── main.yml               # Role metadata
│   └── argument_specs.yml     # Argument validation (CoP §3.1.20)
├── tasks/
│   ├── main.yml               # Main orchestration
│   ├── assert.yml             # Variable validation
│   ├── install.yml            # Package installation
│   ├── configure.yml          # Credential and directory setup
│   ├── certificates.yml       # Certificate issuance
│   ├── renewal.yml            # Auto-renewal configuration
│   ├── validate.yml           # Dry-run testing
│   ├── revoke.yml             # Certificate revocation
│   └── remove.yml             # Cleanup and removal
├── templates/
│   ├── credentials/           # DNS provider credential templates
│   ├── certbot-renewal.*      # Systemd timer and service templates
│   └── certbot-renew-cron.j2  # Cron renewal template
└── vars/
    ├── main.yml               # Internal variables
    ├── debian_12.yml           # Debian 12 specific
    └── ubuntu_24.04.yml       # Ubuntu 24.04 specific
```

### Supported Platforms 🌍
| OS Family | Version | Status |
|-----------|---------|--------|
| Debian | 12 (Bookworm) | ✅ Full Support |
| Ubuntu | 24.04 (Noble) | ✅ Full Support |
