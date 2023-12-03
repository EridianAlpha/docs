# Ubuntu Desktop







### Dual Monitor Configuration

* [https://github.com/AdnanHodzic/displaylink-debian](https://github.com/AdnanHodzic/displaylink-debian)
  * Press "Y" to everything that it asks to install
  * Last step will ask you to reboot the system

```bash
cd ~/Downloads
git clone https://github.com/AdnanHodzic/displaylink-debian.git
cd displaylink-debian/ && sudo ./displaylink-debian.sh
```

* Add settings to allow monitor to be detected
  * [https://support.displaylink.com/knowledgebase/articles/1181623-displaylink-ubuntu-driver-after-recent-x-upgrades](https://support.displaylink.com/knowledgebase/articles/1181623-displaylink-ubuntu-driver-after-recent-x-upgrades)
  * [https://github.com/AdnanHodzic/displaylink-debian/issues/469](https://github.com/AdnanHodzic/displaylink-debian/issues/469)

```bash
xrandr --setprovideroutputsource 1 0
xrandr --setprovideroutputsource 2 0
xrandr --setprovideroutputsource 3 0
xrandr --setprovideroutputsource 4 0
```

```bash
sudo vim /usr/share/X11/xorg.conf.d/20-displaylink.conf
```

{% code title="/usr/share/X11/xorg.conf.d/20-displaylink.conf" %}
```
Section "OutputClass"
	Identifier "DisplayLink"
	MatchDriver "evdi"
	Driver "modesetting"
	Option  "AccelMethod" "none"
EndSection

Section "Device"
  Identifier "DisplayLink"
  Driver "modesetting"
  Option "PageFlip" "false"
EndSection
```
{% endcode %}

* Restart to pick up config changes

```bash
sudo shutdown -r now
```



