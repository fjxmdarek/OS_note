# OS_cmd_note

# host ssh:

```bash
systemctl start ssh
```

# Enable GRUB Boot Menu

Edit the GRUB config file:

```bash
sudo vim /etc/default/grub
```

Change the following two lines:

```
GRUB_TIMEOUT_STYLE=hidden   →   GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=0              →   GRUB_TIMEOUT=30
```

Save and exit (`:wq`), then update GRUB and reboot:

```bash
sudo update-grub
sudo reboot
```

On reboot, a 30-second menu will appear. Select **Advanced options for Ubuntu** to choose your kernel (e.g. `6.8.0-cs4118`).

---

# Optimize Kernel Compilation Time

Strip unused modules from `.config` to speed up builds (only need to do this once):

```bash
cp .config .config.backup
yes '' | make localmodconfig
```

Verify `CONFIG_BLK_DEV_LOOP` is still `y`, then rebuild and install as usual.

---

# Daily Workflow (After Editing `.c` Files)

If you **only changed `.c` files** (no `.h` headers, no config changes):

```bash
make -j$(nproc)
sudo make install
sudo reboot
```

No need to re-run `make modules_install` or redo any configuration steps.
