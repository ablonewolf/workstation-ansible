# Workstation Bootstrap using Ansible + Chezmoi

This repository contains a reproducible Linux workstation bootstrap setup.

The goal is to restore a fresh operating system installation with minimal manual intervention.

The setup combines:

- Ansible → installs packages, themes, services, development tools
- Chezmoi → restores configuration files and dotfiles

This architecture separates responsibilities cleanly.

| Responsibility | Tool |
|---|---|
| System provisioning | Ansible |
| User configuration | Chezmoi |

---

# Philosophy

This repository is designed around the following principles:

- Reproducible workstation setup
- Minimal post-install manual work
- Manifest-driven package restoration
- Separation between packages and configuration
- Cross-distro adaptability
- Idempotent restoration
- Easy future maintenance

---

# What this setup restores

## Official packages

Installed using the native package manager.

Examples:

- pacman
- dnf
- apt

Current implementation primarily targets:

- CachyOS
- Arch Linux

Package manifest:

```text
pkglist-official.txt
```

---

## AUR packages

Installed using:

```text
paru
```

Manifest:

```text
pkglist-aur.txt
```

---

## Flatpak applications

Installed from Flathub.

Manifest:

```text
pkglist-flatpak.txt
```

---

## KDE Themes

Themes are cloned from Git repositories and installed automatically.

Theme definitions:

```text
themes.yml
```

---

## Cursor visibility fixes

Some cursor themes may be hidden from KDE System Settings.

This repository automatically patches:

```text
/usr/share/icons/Adwaita/index.theme
```

to ensure the Adwaita cursor appears in the settings panel.

---

## Development tools

The setup also installs developer tooling if missing:

- SDKMAN
- uv
- rustup
- nvm
- pyenv

---

## Services

The setup also enables/configures:

- Docker
- PostgreSQL
- cronie

---

## Dotfiles

Personal configuration is restored using:

```text
chezmoi
```

Examples:

- .bashrc
- .profile
- git config
- KDE config
- cursor settings
- shell aliases
- SSH config
- terminal settings

---

# Repository Structure

```text
workstation-ansible/
├── inventory.ini
├── local.yml
├── pkglist-official.txt
├── pkglist-aur.txt
├── pkglist-flatpak.txt
├── themes.yml
└── README.md
```

---

# How it works

The restoration flow:

```text
Fresh OS install
→ Install git + ansible
→ Clone repository
→ Run Ansible playbook
→ Install packages
→ Install themes
→ Configure services
→ Apply chezmoi dotfiles
→ Workstation restored
```

---

# Requirements

## Minimum requirements

Install:

- git
- ansible

---

## Arch Linux / CachyOS

```bash
sudo pacman -S --needed git ansible
```

---

## Fedora

```bash
sudo dnf install git ansible
```

---

## Ubuntu / Debian

```bash
sudo apt install git ansible
```

---

# Running the bootstrap

Clone the repository:

```bash
git clone <your-repository-url>
cd workstation-ansible
```

Run the playbook:

```bash
ansible-playbook -i inventory.ini local.yml --ask-become-pass
```

---

# Updating package manifests

The commands differ depending on the Linux distribution and package manager.

---

# Arch Linux / CachyOS

## Official packages

```bash
pacman -Qqen > pkglist-official.txt
```

Explanation:

- `-Q` → query installed packages
- `-q` → output only package names
- `-e` → explicitly installed packages
- `-n` → native packages (official repositories)

---

## AUR packages

```bash
pacman -Qqem > pkglist-aur.txt
```

Explanation:

- `-m` → foreign packages (AUR/manual installs)

---

## Flatpak applications

```bash
flatpak list --app --columns=application > pkglist-flatpak.txt
```

---

# Fedora

Fedora does not separate packages like AUR.

## Official packages

```bash
dnf repoquery --userinstalled --qf "%{name}" \
| sort -u > pkglist-official.txt
```

Alternative:

```bash
dnf history userinstalled \
| sort -u > pkglist-official.txt
```

---

## Flatpak applications

```bash
flatpak list --app --columns=application > pkglist-flatpak.txt
```

---

# Ubuntu / Debian

Ubuntu/Debian systems use APT.

## Official packages

```bash
apt-mark showmanual | sort -u > pkglist-official.txt
```

This exports manually installed packages instead of the full dependency tree.

---

## Flatpak applications

```bash
flatpak list --app --columns=application > pkglist-flatpak.txt
```

---

# KDE Themes

Themes are defined in:

```yaml
themes:
  - name: ExampleTheme
    repo: https://github.com/example/theme.git
    dest: ExampleTheme
    install_command: ./install.sh
```

Ansible will:

1. Clone the repositories
2. Run their install scripts
3. Restore theme selection using chezmoi

---

# PostgreSQL

The playbook:

- initializes PostgreSQL if needed
- enables the PostgreSQL service
- sets the password for the `postgres` user

Current development credentials:

```text
username: postgres
password: postgres
```

These credentials are intended only for local development.

---

# Why use Chezmoi instead of Ansible for dotfiles?

Ansible is excellent for system provisioning.

Chezmoi is optimized specifically for:

- dotfile management
- templating
- secrets
- OS-aware configuration
- idempotent user configuration

Using both tools together creates cleaner separation of concerns.

---

# Why not back up SDKMAN/Rust/NVM directories directly?

Those directories contain:

- caches
- downloaded binaries
- temporary state
- architecture-specific artifacts

Instead, this setup reinstalls tools cleanly.

This makes the setup:

- smaller
- portable
- maintainable
- cross-machine friendly

---

# Cross-platform notes

The current implementation targets:

- CachyOS
- Arch Linux

However, the overall architecture works for any Linux distribution.

To adapt this repository:

## Fedora

Replace:

- pacman tasks
- paru tasks

with:

- dnf tasks

---

## Ubuntu / Debian

Replace:

- pacman tasks
- AUR tasks

with:

- apt tasks

The overall architecture remains unchanged.

---

# Recommended workflow

Whenever packages change:

## Arch/CachyOS

```bash
pacman -Qqen > pkglist-official.txt
pacman -Qqem > pkglist-aur.txt
flatpak list --app --columns=application > pkglist-flatpak.txt
```

---

## Fedora

```bash
dnf repoquery --userinstalled --qf "%{name}" \
| sort -u > pkglist-official.txt

flatpak list --app --columns=application > pkglist-flatpak.txt
```

---

## Ubuntu/Debian

```bash
apt-mark showmanual | sort -u > pkglist-official.txt

flatpak list --app --columns=application > pkglist-flatpak.txt
```

---

Commit and push:

```bash
git add .
git commit -m "Update workstation manifests"
git push
```

---

# Important Notes

This repository is intended for:

- development workstations
- personal Linux setups
- reproducible desktop environments

It is NOT intended for:

- production servers
- hardened enterprise environments
- multi-user infrastructure orchestration

---

# Future Improvements

Possible future improvements:

- Split playbook into Ansible roles
- Multi-distro support
- Secret management using Ansible Vault
- Host-specific variables
- CI validation
- Theme version pinning
- Automatic package cleanup
- Systemd user services
- Wayland/X11-specific logic
- Per-host overlays

---

# License

Personal-use repository.

Modify as needed.
