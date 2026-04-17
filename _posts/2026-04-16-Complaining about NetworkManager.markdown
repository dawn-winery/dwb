---
layout: post
title:  "Complaining about NetworkManager"
date:   2026-04-16 22:39:00 -0600 (replace date and timezone respecively)
categories: dwb networkmanager linux
---
I hate NetworkManager.

Is this just a skill issue and NetworkManager works fine for you? Probably. But I'm going to complain anyways.

1. It has died on me multiple times, including one time (using the iwd backend instead of wpa_supplicant) where nmcli refused to connect due to some issue with Secrets, and using iwctl directly would just error with no explanation. Uninstalling NetworkManager fixed iwd instantly.
2. NetworkManager does not support DNS-over-HTTPS with the default setup, meaning your ISP can see any domain name you connect to
3. It does not support setting a connection to WPA2/3, you have to use either WPA1/2 or WPA3-Only.
4. If you set a connection to WPA3-Only, on the next reboot the connection will be renamed to have a 1 at the end, and connecting to it will fail, so you must connect again and enter the password like its a brand new network. I reproduced this issue across 3 devices, many different distros, and 2 different networks
5. It does not have any meaningful additional features. The most notable one is support for many VPN protocols, but for Wireguard, the most commonly used one, support is baked into the Linux kernel.

In summary, NetworkManager is a wrapper for tools that are already built in to your system, that actively breaks the tools it wraps, and has less features than them in many cases.


## Recommended alternatives:

Either iwd (recommended) or wpa_supplicant, with systemd-networkd and systemd-resolved

Note that systemd-resolved is minimal and for more features like DNS-over-HTTPS you will need something like [Blocky](https://0xerr0r.github.io/blocky/latest/) instead.

## Switching to iwd

Here is how to make the switch on Arch Linux:
```sh
sudo pacman -S iwd
sudo pacman -Rns NetworkManager
sudo systemctl enable --now iwd systemd-networkd systemd-resolved

sudo echo """
[Network]
EnableIPv6=true

[Settings]
AutoConnect=true
""" > /etc/iwd/main.conf
```

And here is a NixOS module for it:
```nix
{
  networking.networkmanager.enable = false;
  networking.useNetworkd = true;
  services.resolved.enable = true;
  systemd.network.enable = true;

  networking.wireless.iwd.enable = true;
  networking.wireless.iwd.settings = {
    Network = {
      EnableIPv6 = true;
    };
    Settings = {
      AutoConnect = true;
    };
  };
}
```

Note that most Network applets, such as those of KDE Plasma, GNOME, and Noctalia, do not support iwd (or wpa_supplicant). Unless your shell supports it (eg. DankMaterialShell), you will need to use `iwctl` in the command line, or install a standalone applet like `aur/iwqt`

\- Sofia