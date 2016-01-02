## Preventing OS X from phoning home to Cupertino

### Why

* When you're pentesting, you want your machine to stay absolutely quiet.
* When you're booked into a public wifi, eavesdroppers may glean personal information from traffic inadvertantly generated by your machine. (Some of the hardcoded URLs use unencrypted http.)
* If you're a dissident, your whereabouts may be revealed and you may not even know it.

### How

* I searched the entire OS X Mavericks base installation for hardcoded URLs and IP addresses. The domain names used in the URLs are hardwired to 127.0.0.1 in `/etc/hosts`. The IP addresses are natted to 127.0.0.1 in `/etc/pf.conf`. A number of LaunchAgents, LaunchDaemons, UserEventPlugins plus all Dashboard Widgets should be disabled by moving them to, say, `/root/disabled/`. Those are listed in `disabled-services`.
* Edit `/System/Library/LaunchDaemons/com.apple.mDNSResponder.plist` and add the undocumented option `-NoMulticastAdvertisements`.
* Disable Dashboard: `defaults write com.apple.dashboard mcx-disabled -boolean YES && killall Dock`
* Disable some IPv6 features of dubious merit in `/etc/sysctl.conf`.

### Caution

* This is for Mavericks, not Yosemite.
* It will yield a machine that stays quiet when connected to a network but at the expense of convenience features like push notifications. Also, the log files will show a few error messages because of unavailable services.
* Several services regularly contact www.apple.com to check for network connectivity. Thus, www.apple.com is blacklisted in `/etc/hosts`. Comment out manually whenever you want to browse that website.
* When connected to a wifi, the machine will regularly send EAPOL packets which cannot be disabled because OS X cannot packet filter on Layer 2. (`pfctl(8)` only filters on layer 3 and upwards and `ipfw(8)` doesn't work either.)
* OS X stores wifi passwords in NVRAM. This is apparently used by Internet Recovery. Thus, whenever your machine is stolen or lent to someone else, consider your wifi passwords compromised, regardless if the disk was encrypted. It seems that FindMyMacd clears the NVRAM if the machine was stolen but this is not safe: FindMyMacd itself is apparently controlled by NVRAM variables and a thief may change these to disable it. Wifi passwords can be retrieved from NVRAM like this:
```
/usr/libexec/airportd readNVRAM

/usr/sbin/nvram 36C28AB5-6566-4C50-9EBD-CBB920F83843:current-network
/usr/sbin/nvram 36C28AB5-6566-4C50-9EBD-CBB920F83843:preferred-networks
/usr/sbin/nvram 36C28AB5-6566-4C50-9EBD-CBB920F83843:preferred-count
```

### Ideas for further hacks

* Use a proxy on the local machine to MitM or spoof traffic to Cupertino.
