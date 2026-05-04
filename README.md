# Ansible Role: Certbot

|Source|Version|Tests|License|
|------|-------|-------|-------|
|[![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-certbot)|[![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-certbot)](https://github.com/grzegorzfranus/ansible-role-certbot/releases)|[![tests](https://github.com/grzegorzfranus/ansible-role-certbot/actions/workflows/test-and-validation.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-certbot/actions)|[![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE)|

This Ansible role installs and configures Certbot for automated SSL/TLS certificate management with Let's Encrypt. It supports multiple validation methods, DNS providers, web server integrations, wildcard certificates, automatic renewal, and certificate revocation.

## ✨ Features

- 🔐 **SSL/TLS Automation**: Automated certificate issuance from Let's Encrypt
- 🌐 **HTTP Validation**: Standalone, webroot, Nginx, and Apache authenticators
- 🌍 **DNS Validation**: Cloudflare, Route53 (AWS), and DigitalOcean providers
- 🃏 **Wildcard Certificates**: Full wildcard support via DNS-01 challenge
- 🔄 **Auto-Renewal**: Systemd timer or cron-based automatic renewal
- 🪝 **Renewal Hooks**: Pre, post, and deploy hooks for service management
- 🧪 **Staging Support**: Test with Let's Encrypt staging to avoid rate limits
- ❌ **Certificate Revocation**: Revoke with RFC 5280 reason codes
- 🔑 **Key Types**: ECDSA (secp256r1, secp384r1, secp521r1) and RSA (2048-4096)
- 🛡️ **Secure Credentials**: `no_log`, `0600` permissions, Vault integration
- 📋 **Argument Validation**: `meta/argument_specs.yml` for Ansible-native checks (CoP §3.1.20)
- 🧪 **Dry-Run Testing**: Validate renewal configuration without modifying certificates

## 🎯 Architecture

Certbot automates the ACME protocol to obtain trusted certificates from Let's Encrypt:

```
Your Server ←→ Let's Encrypt ACME Server
    ↓                    ↓
Challenge            Validation
(HTTP-01 or DNS-01)  (Domain ownership proof)
    ↓                    ↓
Certificate Issued → Auto-Renewal Timer
```

### Validation Methods

| Method | Challenge | Wildcard | Port Required | Web Server |
|--------|-----------|----------|---------------|------------|
| Standalone | HTTP-01 | ❌ | 80 (temp) | None needed |
| Webroot | HTTP-01 | ❌ | 80 (existing) | Any (running) |
| Nginx | HTTP-01 | ❌ | 80 (existing) | Nginx (running) |
| Apache | HTTP-01 | ❌ | 80 (existing) | Apache (running) |
| DNS | DNS-01 | ✅ | None | None needed |

### DNS Providers

| Provider | Plugin | Authentication |
|----------|--------|---------------|
| Cloudflare | `dns-cloudflare` | API Token (Zone:DNS:Edit) |
| Route53 (AWS) | `dns-route53` | Access Key + Secret Key |
| DigitalOcean | `dns-digitalocean` | API Token (read/write) |

## 📋 Requirements

- **Ansible**: 2.16 or higher
- **Network**: Internet access to Let's Encrypt ACME servers
- **Privileges**: sudo/root access on target hosts
- **Port 80**: Required for HTTP-01 validation (not for DNS-01)

### Supported operating systems

| OS Family | Version | Status |
|-----------|---------|--------|
| Debian | 12 (Bookworm) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Ubuntu | 24.04 (Noble) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |

### Ansible version

Ansible >= 2.16

### Python version

Python >= 3.9

### Setup module

The role uses facts gathered by Ansible on the remote host. If you disable the Setup module in your playbook, the role will not work properly.

### Root access

This role requires root access for package installation and certificate management. Make sure you are using a user with root privileges.

## 🚀 Quick Start

### 1. Basic HTTP Standalone Certificate

```yaml
---
- name: Obtain SSL Certificate
  hosts: webservers
  become: true
  roles:
    - role: grzegorzfranus.certbot
      vars:
        certbot_email: "admin@example.com"
        certbot_domains:
          - name: "example.com"
            domains:
              - "example.com"
              - "www.example.com"
```

### 2. Nginx Integration (Zero-Downtime)

```yaml
---
- name: SSL with Nginx Auto-Install
  hosts: webservers
  become: true
  roles:
    - role: grzegorzfranus.certbot
      vars:
        certbot_email: "admin@example.com"
        certbot_http_method: "nginx"
        certbot_domains:
          - name: "example.com"
            domains:
              - "example.com"
              - "www.example.com"
```

### 3. DNS Wildcard Certificate (Cloudflare)

```yaml
---
- name: Wildcard SSL via Cloudflare DNS
  hosts: webservers
  become: true
  roles:
    - role: grzegorzfranus.certbot
      vars:
        certbot_email: "admin@example.com"
        certbot_validation_method: "dns"
        certbot_dns_provider: "cloudflare"
        certbot_cloudflare_api_token: "{{ vault_cloudflare_token }}"
        certbot_domains:
          - name: "example.com"
            domains:
              - "*.example.com"
              - "example.com"
```

### 4. Run the playbook

```bash
ansible-playbook -i inventory certbot-setup.yml
```

## ⚙️ Configuration

### Default Configuration

The role comes with production-ready defaults:

```yaml
# ACME settings
certbot_email: ""                        # Must be set
certbot_environment: "production"        # staging or production

# Validation
certbot_validation_method: "http"        # http or dns
certbot_http_method: "standalone"        # standalone, webroot, nginx, apache

# Renewal
certbot_auto_renewal_enabled: true
certbot_renewal_method: "systemd"

# Key settings
certbot_key_type: "ecdsa"
certbot_elliptic_curve: "secp384r1"
```

## 📊 Variables

### Installation Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_state` | Role state: `present` to install, `absent` to remove | `"present"` |
| `certbot_upgrade` | Upgrade Certbot to latest available version | `false` |

### ACME Account Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_email` | Email for Let's Encrypt account registration and notifications | `""` |
| `certbot_agree_tos` | Agree to ACME Terms of Service | `true` |
| `certbot_environment` | ACME environment: `staging` (testing) or `production` (trusted certs) | `"production"` |

### Validation Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_validation_method` | Default validation: `http` or `dns` | `"http"` |
| `certbot_http_method` | HTTP method: `standalone`, `webroot`, `nginx`, `apache` | `"standalone"` |
| `certbot_webroot_path` | Document root for webroot validation | `"/var/www/html"` |

### Domain Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_domains` | List of certificate definitions with domains and optional per-cert overrides | `[]` |

### DNS Provider Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_dns_provider` | DNS provider: `cloudflare`, `route53`, `digitalocean` | `"cloudflare"` |
| `certbot_dns_propagation_seconds` | DNS propagation wait time (seconds) | `10` |
| `certbot_cloudflare_api_token` | Cloudflare API token (Zone:DNS:Edit permission) | `""` |
| `certbot_route53_access_key_id` | AWS access key ID for Route53 | `""` |
| `certbot_route53_secret_access_key` | AWS secret access key for Route53 | `""` |
| `certbot_digitalocean_token` | DigitalOcean API token (read/write scope) | `""` |
| `certbot_dns_credentials_path` | Directory for credential files | `"/etc/letsencrypt/credentials"` |

### Certificate Path Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_config_dir` | Certbot configuration directory | `"/etc/letsencrypt"` |
| `certbot_work_dir` | Certbot working directory | `"/var/lib/letsencrypt"` |
| `certbot_log_dir` | Certbot log directory | `"/var/log/letsencrypt"` |

### Key Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_key_type` | Key type: `rsa` or `ecdsa` | `"ecdsa"` |
| `certbot_rsa_key_size` | RSA key size (2048-4096) | `4096` |
| `certbot_elliptic_curve` | ECDSA curve: `secp256r1`, `secp384r1`, `secp521r1` | `"secp384r1"` |

### Auto-Renewal Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_auto_renewal_enabled` | Enable automatic certificate renewal | `true` |
| `certbot_renewal_method` | Renewal method: `systemd` or `cron` | `"systemd"` |
| `certbot_renewal_schedule` | Systemd timer OnCalendar schedule | `"*-*-* 02:30:00"` |
| `certbot_renewal_cron_schedule` | Cron schedule expression | `"30 2 * * *"` |
| `certbot_renewal_pre_hook` | Command to run before renewal | `""` |
| `certbot_renewal_post_hook` | Command to run after renewal attempt | `""` |
| `certbot_renewal_deploy_hook` | Command to run after successful renewal | `""` |

### Validation & Testing

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_dry_run_test` | Run renewal dry-run test after setup | `true` |

### Certificate Revocation

| Variable | Description | Default |
|----------|-------------|---------|
| `certbot_revoke_certs` | List of certificate names to revoke | `[]` |
| `certbot_revoke_reason` | Reason: `unspecified`, `keycompromise`, `affiliationchanged`, `superseded`, `cessationofoperation` | `"unspecified"` |
| `certbot_revoke_delete_after` | Delete certificate files after revocation | `true` |

## 📌 Role Properties

| Property | Value | Description |
|----------|-------|-------------|
| **Idempotent** | ✅ Yes | Existing certificates are detected and skipped. Renewal config is updated in place. |
| **Atomic** | ❌ No | Partial failure possible. Use `certbot_state: absent` to clean up. |
| **Check Mode** | ✅ Supported | Most tasks work in check mode. Certificate issuance and renewal are skipped. |
| **Diff Mode** | ✅ Supported | Template tasks support diff mode for change preview. |

## 📤 Role Output

This role does not set any public output facts. All internal facts use the `__certbot_` double-underscore prefix and are not part of the public interface.

## 🔍 Verification

After deployment, verify certificates are working:

### Check Certificate Status

```bash
# List all certificates
sudo certbot certificates

# Check specific certificate
sudo openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -text -noout

# Verify certificate chain
sudo openssl verify -CAfile /etc/letsencrypt/live/example.com/chain.pem \
  /etc/letsencrypt/live/example.com/cert.pem
```

### Test Renewal

```bash
# Dry-run renewal test
sudo certbot renew --dry-run

# Check renewal timer
sudo systemctl status certbot-renewal.timer
sudo systemctl list-timers | grep certbot
```

## 🛡️ Security Considerations

- ✅ **Credential Protection**: DNS credentials stored with `0600` permissions in isolated directory
- ✅ **Secret Masking**: All credential tasks use `no_log: true` to prevent leakage
- ✅ **Ansible Vault**: Use Vault for `certbot_cloudflare_api_token`, `certbot_route53_*`, `certbot_digitalocean_token`
- ✅ **Staging Testing**: Use `certbot_environment: staging` to test without rate limits
- ✅ **Key Security**: ECDSA with secp384r1 by default (stronger, faster than RSA)

### Secure Credential Example

```yaml
# In group_vars/all/vault.yml (encrypted with ansible-vault)
vault_cloudflare_token: "your-api-token-here"
vault_certbot_email: "admin@example.com"

# In playbook
certbot_email: "{{ vault_certbot_email }}"
certbot_cloudflare_api_token: "{{ vault_cloudflare_token }}"
```

## 🔧 Troubleshooting

### Certificate Issues

```bash
# Check Certbot logs
sudo tail -f /var/log/letsencrypt/letsencrypt.log

# Test HTTP-01 challenge manually
curl -I http://example.com/.well-known/acme-challenge/test

# Verify port 80 is available (standalone)
sudo ss -tlnp | grep :80
```

### Renewal Issues

```bash
# Check timer status
sudo systemctl status certbot-renewal.timer
sudo journalctl -u certbot-renewal.service

# Check cron job
cat /etc/cron.d/certbot-renew

# Manual renewal test
sudo certbot renew --dry-run --verbose
```

### DNS Validation Issues

```bash
# Check credential file permissions
ls -la /etc/letsencrypt/credentials/

# Test DNS propagation manually
dig TXT _acme-challenge.example.com

# Increase propagation wait time
certbot_dns_propagation_seconds: 60
```

### Uninstall

```yaml
---
- name: Remove Certbot
  hosts: webservers
  become: true
  roles:
    - role: grzegorzfranus.certbot
      vars:
        certbot_state: absent
```

## 📁 File Structure

```
ansible-role-certbot/
├── CHANGELOG.md              # Version history and changes
├── LICENSE                   # Apache-2.0 license
├── README.md                # This documentation file
├── defaults/
│   └── main.yml             # Default configuration variables
├── files/                   # Static files (placeholder)
├── handlers/
│   └── main.yml             # Service restart and reload handlers
├── meta/
│   ├── main.yml             # Role metadata and Galaxy information
│   └── argument_specs.yml   # Ansible-native argument validation (CoP §3.1.20)
├── tasks/
│   ├── main.yml             # Main task orchestration
│   ├── assert.yml           # Variable validation and system checks
│   ├── install.yml          # Package and plugin installation
│   ├── configure.yml        # Directory and credential setup
│   ├── certificates.yml     # Certificate issuance
│   ├── renewal.yml          # Auto-renewal configuration
│   ├── validate.yml         # Dry-run testing
│   ├── revoke.yml           # Certificate revocation
│   └── remove.yml           # Cleanup and removal
├── templates/
│   ├── credentials/
│   │   ├── cloudflare.ini.j2      # Cloudflare credential template
│   │   ├── route53.ini.j2         # AWS Route53 credential template
│   │   └── digitalocean.ini.j2    # DigitalOcean credential template
│   ├── certbot-renewal.timer.j2   # Systemd timer unit
│   ├── certbot-renewal.service.j2 # Systemd service unit
│   └── certbot-renew-cron.j2      # Cron renewal template
└── vars/
    ├── main.yml             # Internal role variables
    ├── debian_12.yml        # Debian 12 specific variables
    └── ubuntu_24.04.yml     # Ubuntu 24.04 specific variables
```

## 🏷️ Tags

- `always` — Tasks that always run (variable loading and validation)
- `setup` — Setup and configuration tasks
- `init` — Initial environment setup and variable loading
- `validate` — Variable validation and system checks
- `install` — Package installation tasks
- `configure` — Service and credential configuration tasks
- `certificates` — Certificate issuance tasks
- `renewal` — Auto-renewal configuration tasks
- `revoke` — Certificate revocation tasks

## Example Playbook

```yaml
---
# Example 1: HTTP Standalone with Staging
- name: Test SSL Certificate (Staging)
  hosts: staging_servers
  become: true
  vars:
    certbot_email: "admin@example.com"
    certbot_environment: "staging"
    certbot_domains:
      - name: "staging.example.com"
        domains:
          - "staging.example.com"
  roles:
    - role: grzegorzfranus.certbot

# Example 2: Nginx with Multiple Domains
- name: Production SSL with Nginx
  hosts: web_servers
  become: true
  vars:
    certbot_email: "{{ vault_certbot_email }}"
    certbot_http_method: "nginx"
    certbot_renewal_deploy_hook: "systemctl reload nginx"
    certbot_domains:
      - name: "example.com"
        domains:
          - "example.com"
          - "www.example.com"
      - name: "api.example.com"
        domains:
          - "api.example.com"
  roles:
    - role: grzegorzfranus.certbot

# Example 3: DNS Wildcard with Cloudflare
- name: Wildcard Certificate via Cloudflare
  hosts: all
  become: true
  vars:
    certbot_email: "{{ vault_certbot_email }}"
    certbot_validation_method: "dns"
    certbot_dns_provider: "cloudflare"
    certbot_cloudflare_api_token: "{{ vault_cloudflare_token }}"
    certbot_dns_propagation_seconds: 30
    certbot_domains:
      - name: "example.com"
        domains:
          - "*.example.com"
          - "example.com"
  roles:
    - role: grzegorzfranus.certbot

# Example 4: Mixed Validation Methods
- name: Mixed HTTP and DNS Certificates
  hosts: app_servers
  become: true
  vars:
    certbot_email: "admin@example.com"
    certbot_validation_method: "http"
    certbot_http_method: "webroot"
    certbot_webroot_path: "/var/www/html"
    certbot_dns_provider: "route53"
    certbot_route53_access_key_id: "{{ vault_aws_access_key }}"
    certbot_route53_secret_access_key: "{{ vault_aws_secret_key }}"
    certbot_domains:
      - name: "app.example.com"
        domains:
          - "app.example.com"
        validation_method: "http"
        http_method: "webroot"
      - name: "internal.example.com"
        domains:
          - "*.internal.example.com"
        validation_method: "dns"
  roles:
    - role: grzegorzfranus.certbot

# Example 5: Certificate Revocation
- name: Revoke Compromised Certificate
  hosts: web_servers
  become: true
  vars:
    certbot_email: "admin@example.com"
    certbot_revoke_certs:
      - "old.example.com"
      - "compromised.example.com"
    certbot_revoke_reason: "keycompromise"
    certbot_revoke_delete_after: true
  roles:
    - role: grzegorzfranus.certbot

# Example 6: Cron-Based Renewal with Hooks
- name: SSL with Cron Renewal and Apache
  hosts: legacy_servers
  become: true
  vars:
    certbot_email: "admin@example.com"
    certbot_http_method: "apache"
    certbot_renewal_method: "cron"
    certbot_renewal_cron_schedule: "0 3 * * 1"
    certbot_renewal_deploy_hook: "systemctl reload apache2"
    certbot_domains:
      - name: "legacy.example.com"
        domains:
          - "legacy.example.com"
  roles:
    - role: grzegorzfranus.certbot
```

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`.
- Make your changes with clear, descriptive commit messages.
- Ensure your code passes all Molecule and lint tests.
- Submit a pull request describing your changes and the motivation.
- For major changes, please open an issue first to discuss what you would like to change.

If you have questions or suggestions, feel free to open an issue or contact the author via GitHub.

## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).