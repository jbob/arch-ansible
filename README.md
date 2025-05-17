# 🛠️ Encrypted Arch Linux Setup with Ansible & Btrfs

This Ansible playbook automates the installation of Arch Linux with full disk encryption and a Btrfs file system. A second stage handles essential post-install tasks like package installation, `sudo` configuration, and setting up `trizen` as an AUR helper.

---

## 🚀 Usage Guide

### 🔧 1. Boot the Target Device

* Boot the target device from a prepared Arch Linux USB stick.
* Ensure network connectivity:

  * For **wired connections**, this should work automatically.
  * For **Wi-Fi**, refer to the [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide#Connect_to_the_internet).

### 🧪 2. In the Live Environment (on the target device)

Run the following:

```bash
passwd                  # Set root password
ip addr | grep inet     # Get IP address
lsblk                   # Identify target disk (e.g. /dev/nvme0n1 or /dev/sda)
```

---

### 👤 3. On Your Ansible Host (a different machine)

Make sure Ansible and `python-passlib` are installed.

#### 🔹 SSH Setup

1. In the `hosts` file, set the desired hostname (e.g., `archbox`).

2. Add the machine to your `~/.ssh/config`:

   ```ssh
   Host archbox
     HostName 192.168.x.x  # Replace with device IP
     User root
   ```

3. Copy your SSH key to the target machine:

   ```bash
   ssh-copy-id archbox
   ```

---

#### 🔹 Adjust Configuration

Edit `group_vars/all.yml`:

* `install_drive`: Target disk (e.g. `/dev/nvme0n1`)
* `luks_password`: LUKS encryption password
* `user_password`: Initial user password
* `username`: Your preferred username
* `timezone` & `keymap`: Set your locale
* `packages`: List of packages to install
* `boot_disk_size`, `swap_size`, `grub_cmdline`: Adjust as needed

---

### ▶️ 4. Run the Bootstrap Phase

This step partitions the disk, sets up LUKS and Btrfs, installs the base system, and configures initial settings.

```bash
ansible-playbook -i hosts bootstrap.yml
```

> ⚠️ **Note**: This step is destructive and can only be safely run **once**. If it fails or needs to be re-run, you’ll need to restart the installation from scratch.

---

### 🔁 5. Finalize Setup

1. Reboot the target device from its internal drive.

2. Check its new IP address. If it changed, update `~/.ssh/config`.

3. Ensure SSH access still works:

   ```bash
   ssh archbox
   ```

   If needed, clear the relevant lines from `~/.ssh/known_hosts`.

4. Run the second-stage configuration:

   ```bash
   ansible-playbook -i hosts step2.yml
   ```

> This step is **safe to re-run** and can be used for iterative tweaks and package additions.

---

## ✅ Done!

You now have a fully encrypted Arch Linux installation with Btrfs, configured with your chosen settings and ready for further customization.

