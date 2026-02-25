# OS_note

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
