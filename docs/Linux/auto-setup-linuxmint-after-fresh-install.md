# Automating Your Linux Mint Setup After a Fresh Install

Setting up a fresh Linux Mint installation can be time-consuming, especially when you want to replicate your perfect development environment. This guide will show you how to automate the entire process using Ansible and configuration backups, so you can go from a fresh install to a fully configured system in minutes.

## Why Automate Your Setup?

Whether you're setting up a new machine, recovering from a system failure, or just want to maintain consistency across multiple computers, automation offers several key benefits:

- **Time Savings:** What normally takes hours can be done in minutes
- **Consistency:** Identical setup across all your machines
- **Documentation:** Your setup becomes self-documenting
- **Recovery:** Quick recovery from system failures
- **Reproducibility:** Never forget to install that one crucial tool again

## Discovering Your Installed Applications

Before creating your automation setup, you need to identify which applications you've manually installed since the initial OS installation. This helps you build a complete picture of your custom environment.

### Finding APT and .deb Packages

To see all manually installed packages (excluding those that came with the OS):

```bash
comm -23 <(apt-mark showmanual | sort -u) <(gzip -dc /var/log/installer/initial-status.gz | sed -n 's/^Package: //p' | sort -u)
```

**What this does:**
- `apt-mark showmanual` lists all manually installed packages
- `/var/log/installer/initial-status.gz` contains packages from the initial installation
- `comm -23` compares the two lists and shows only packages you installed after setup
- This helps you identify exactly what to include in your Ansible playbook

**Tip:** Save this output to a file for reference:
```bash
comm -23 <(apt-mark showmanual | sort -u) <(gzip -dc /var/log/installer/initial-status.gz | sed -n 's/^Package: //p' | sort -u) > manually-installed-packages.txt
```

### Finding Flatpak Applications

To list all installed Flatpak applications:

```bash
flatpak list --app
```

**What this does:**
- Lists all Flatpak applications installed on your system
- The `--app` flag filters out runtimes and shows only applications
- Use this list to populate the Flatpak section of your Ansible playbook

**Getting more details:**
```bash
# Show application IDs (needed for Ansible)
flatpak list --app --columns=application

# Show with origin (where it was installed from)
flatpak list --app --columns=application,origin
```

### Creating Your Package Inventory

Use these commands to build a comprehensive inventory:

```bash
# Create a directory for your automation files
mkdir -p ~/linux-mint-automation

# Save APT packages
comm -23 <(apt-mark showmanual | sort -u) <(gzip -dc /var/log/installer/initial-status.gz | sed -n 's/^Package: //p' | sort -u) > ~/linux-mint-automation/apt-packages.txt

# Save Flatpak apps
flatpak list --app --columns=application > ~/linux-mint-automation/flatpak-apps.txt
```

Now you have a clear reference of what needs to be included in your automation setup!

## Overview of the Automation Strategy

This guide uses a three-part approach:

1. **Ansible Playbook** - Automates software installation and system configuration
2. **Configuration Files** - Backs up and restores application settings from `.config`
3. **dconf Backup** - Preserves desktop environment settings (Cinnamon/GNOME)

I store all configurations in a private repository to protect any sensitive information while keeping everything version-controlled and easily accessible.

## Prerequisites

Before you begin, make sure you have:

- A fresh Linux Mint installation
- Terminal access
- An internet connection
- Basic familiarity with the command line

## Step-by-Step Setup Process

### Step 1: Update Your System

First things first—let's make sure your system is up to date:

```bash
sudo apt update
sudo apt upgrade -y
```

**What this does:**
- Updates the package index to get the latest package information
- Upgrades all installed packages to their latest versions
- The `-y` flag automatically confirms all prompts

### Step 2: Install Ansible

Ansible is a powerful automation tool that will handle the bulk of our software installation. Add the official Ansible PPA and install it:

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

**What this does:**
- Adds the official Ansible Personal Package Archive (PPA)
- Refreshes package information to include Ansible packages
- Installs the latest version of Ansible

### Step 3: Create Your Ansible Playbook

Create an Ansible playbook that defines your entire system configuration. This YAML file will automate software installation from multiple sources.

Create a file named `localsetup.yml`:

```yaml
- hosts: localhost
  become: true

  vars:
    # Add any variables here (URLs, versions, etc.)

  tasks:
    # Install prerequisites
    - name: Install prerequisites for Ansible to install .deb via apt module
      apt:
        name:
          - xz-utils
        state: present

    - name: Ensure wget and gpg are installed
      apt:
        name:
          - wget
          - gpg
        state: present

    # Install applications from .deb packages
    - name: Install Google Chrome
      apt:
        deb: https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

    # Add third-party repositories
    - name: Add signing key for Tailscale
      get_url:
        url: https://pkgs.tailscale.com/stable/ubuntu/jammy.noarmor.gpg
        dest: /usr/share/keyrings/tailscale.gpg
        mode: '0644'

    - name: Add Tailscale repository
      apt_repository:
        repo: "deb [signed-by=/usr/share/keyrings/tailscale.gpg] https://pkgs.tailscale.com/stable/ubuntu jammy main"
        filename: tailscale
        state: present

    - name: Update package cache after adding repositories
      ansible.builtin.apt:
        update_cache: yes

    # Install standard packages from Ubuntu/Mint repositories
    - name: Install essential applications
      apt:
        pkg:
          - tailscale
          - git
          - diodon          # Clipboard manager
          - pavucontrol     # PulseAudio volume control
          - guake           # Drop-down terminal
          - vim
          - curl
          - htop
        state: present

    # Install Flatpak applications
    - name: Install Flatpak applications
      community.general.flatpak:
        name:
          - org.gimp.GIMP
          - org.inkscape.Inkscape
        state: present
```

**Understanding the playbook structure:**

- **hosts: localhost** - Runs on your local machine
- **become: true** - Executes tasks with sudo privileges
- **tasks** - List of operations to perform
- **apt module** - Installs packages from repositories or .deb files
- **apt_repository** - Adds third-party repositories
- **flatpak module** - Installs Flatpak applications

**Customization tips:**
- Add more packages to the `pkg` list under "Install essential applications"
- Include additional `.deb` packages using the `deb:` parameter
- Add more third-party repositories following the Tailscale example
- Extend with Flatpak apps, snap packages, or pip packages as needed

### Step 4: Run the Ansible Playbook

Navigate to the directory containing your `localsetup.yml` file and execute the playbook:

```bash
sudo ansible-playbook localsetup.yml --connection=local
```

**What this does:**
- Executes all tasks defined in your playbook
- Installs all specified software automatically
- Configures repositories and signing keys
- Runs with local connection (no SSH required)

**Note:** This may take several minutes to complete, depending on your internet connection and the number of packages being installed. You'll see progress output for each task.

### Step 5: Restore Configuration Files

Application settings are typically stored in the `~/.config` directory. If you have a backup of your configuration files, restore them:

```bash
# Clone your private configuration repository
git clone https://github.com/yourusername/your-config-repo.git
cd your-config-repo

# Copy configuration files to your home directory
cp -r .config/* ~/.config/
```

**What this restores:**
- Application preferences and settings
- Custom keyboard shortcuts
- Editor configurations (VS Code, Vim, etc.)
- Terminal emulator settings
- Any other application-specific configurations

**Tip:** Press `Ctrl + H` in your file manager to show hidden files and folders (those starting with `.`).

**Important configurations to backup:**
- `~/.config/` - Most modern application settings
- `~/.bashrc` or `~/.zshrc` - Shell configuration
- `~/.gitconfig` - Git configuration

### Step 6: Import Desktop Environment Settings

Restore your Cinnamon/GNOME desktop environment settings using dconf. This includes themes, panels, applets, and all desktop preferences.

#### Creating a dconf Backup (do this on your working system first):

```bash
# Export all settings
dconf dump / > my_dconf_backup.conf

# Or export specific paths
dconf dump /org/cinnamon/ > cinnamon_settings.conf
```

#### Restoring dconf Settings:

```bash
# Navigate to your dconf backup location
cd /path/to/your/backups/dconf

# Restore all settings
dconf load / < my_dconf_backup.conf
```

**What this restores:**
- Desktop themes and appearance
- Panel configuration and applets
- Keyboard shortcuts
- Window manager preferences
- Display settings
- Power management settings
- All other desktop environment preferences

**Note:** You may need to log out and log back in for all changes to take effect.

## Advanced Tips and Best Practices

### Maintaining Your Automation Setup

1. **Version Control:** Keep your playbook and configs in a Git repository
2. **Regular Updates:** Update your backup after making configuration changes
3. **Test on VMs:** Test your automation on a virtual machine before using on production
4. **Document Changes:** Add comments to your playbook explaining custom configurations

### Securing Sensitive Information

- Use Ansible Vault to encrypt sensitive data in your playbooks
- Never commit SSH private keys or passwords to repositories
- Use environment variables for sensitive configuration values
- Keep your configuration repository private

### Extending Your Automation

You can extend this setup to include:

```yaml
# Install development tools
- name: Install development tools
  apt:
    pkg:
      - build-essential
      - docker.io
      - python3-pip
      - nodejs
      - npm

# Install VS Code
- name: Add VS Code repository key
  apt_key:
    url: "https://packages.microsoft.com/keys/microsoft.asc"
    state: present

- name: Add VS Code repository
  apt_repository:
    repo: "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
    filename: vscode
    state: present

- name: Install VS Code
  apt:
    name: code
    state: present

# Configure git
- name: Configure git user
  community.general.git_config:
    name: "{{ item.name }}"
    value: "{{ item.value }}"
    scope: global
  loop:
    - { name: 'user.name', value: 'Your Name' }
    - { name: 'user.email', value: 'your.email@example.com' }
```

## Troubleshooting

### Common Issues

**Ansible not found after installation:**
```bash
# Verify installation
ansible --version

# If not found, check PATH
echo $PATH
```

**Permission denied errors:**
- Ensure you're running the playbook with `sudo`
- Check file permissions on your playbook: `chmod 644 localsetup.yml`

**Package conflicts:**
- Run `sudo apt update` before running the playbook
- Check for PPA conflicts: `sudo apt-cache policy <package-name>`

**dconf restore doesn't seem to work:**
- Log out and log back in after restoring
- Verify the backup file format: `head my_dconf_backup.conf`
- Try restoring specific paths instead of all settings

## Conclusion

Automating your Linux Mint setup transforms a tedious manual process into a quick, repeatable procedure. With Ansible handling software installation, configuration file backups preserving application settings, and dconf managing desktop preferences, you can rebuild your perfect development environment in minutes rather than hours.

The time invested in creating these automation scripts pays dividends every time you set up a new machine, recover from a failure, or help a colleague replicate your environment.

## Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [dconf Manual](https://wiki.gnome.org/Projects/dconf)
- [Linux Mint Documentation](https://linuxmint.com/documentation.php)
- [Dotfiles Management](https://dotfiles.github.io/)

## Next Steps

- Customize the playbook with your preferred applications
- Create backups of your current configuration files
- Export your current dconf settings
- Test your automation on a virtual machine
- Set up a private Git repository for your configurations