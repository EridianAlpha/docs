# Ubuntu Desktop

### Settings - Desktop

* Change background image

```bash
cd ~/Downloads
wget https://images2.alphacoders.com/118/thumb-1920-1188102.jpg
gsettings set org.gnome.desktop.background picture-uri file:////home/eridian/Downloads/thumb-1920-1188102.jpg
gsettings set org.gnome.desktop.background picture-uri-dark file:////home/eridian/Downloads/thumb-1920-1188102.jpg
```

* Remove all desktop icons

```bash
gsettings set org.gnome.shell.extensions.ding show-trash false
gsettings set org.gnome.shell.extensions.ding show-home false

gsettings set org.gnome.shell favorite-apps "['org.gnome.Terminal.desktop', 'firefox.desktop']"
```

Install SSH

```bash
sudo apt-get update -y
sudp apt-get install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
```

* Settings
  * Wi-Fi - Airplane mode - `Check`
  * Appearance
    * Appearance - `Dark`
      * Auto-hide the Dock - `Check`
      * Configure dock behavior
        * Show Volumes and Devices - `Uncheck`
        * Show Trash - `Uncheck`
  * Power - Screen Blank - `Never`



* Onscreen keyboard with arrow keys

{% tabs %}
{% tab title="onboard" %}
```bash
sudo apt-get install onboard
```

* Set to favorites
* Onboard settings
  * General
    * Auto-show when editing text - `Uncheck`
    * Show floating icon when Onboard is hidden - `Check`
  * Window
    * Dock to screen edge - `Uncheck`
    * Force window to top - `Check`
{% endtab %}

{% tab title="Raspberry Pi" %}
`onboard` didn't work on a Raspberry pi, so instead just use the default on-screen keyboard in the Settings -> Accessibility menu.
{% endtab %}
{% endtabs %}

* Add system info to top bar
  * [https://www.computernetworkingnotes.com/linux-tutorials/ubuntu-show-cpu-and-memory-usages-in-top-bar.html](https://www.computernetworkingnotes.com/linux-tutorials/ubuntu-show-cpu-and-memory-usages-in-top-bar.html)

```
sudo apt-get install -y indicator-multiload
```

* Open the application directly by searching in the activities search bar so that it auto-starts on system restart
  * `System Load Indicator`
* Edit the preferences
  * Change the top item in the `Indicator items...` list (include the extra spaces to spread out the values)

```
    CPU $(percent(cpu.inuse))    Mem $(size(mem.user))
```

* Change the `System monitor update interval` to `2000`

### Settings - Terminal Preferences

* Change terminal colors
  * Hamburger button in terminal -> Preferences -> Profiles
  * Rename "Unamed" to `Profile1`
  * Colors
    * Uncheck "Use transparency from system theme" and then check `Use transparent background` (set to about `25%`)
    * Check `Show bold text in bright colors`
* Create a `Profile2` that's a clone of `Profile1`
  * This will be easier to work with when it's important to see everything very clearly
  * Colors
    * Uncheck "Use colors from system theme" and select `White on black`
    * Set the background transparency to None

### 🖥️ 🖥️ Dual Monitor Configuration

* Disable secure boot in the BIOS
* [https://github.com/AdnanHodzic/displaylink-debian](https://github.com/AdnanHodzic/displaylink-debian)
  * Press "Y" to everything that it asks to install
  * Last step will ask you to reboot the system

```bash
cd ~/Downloads
git clone https://github.com/AdnanHodzic/displaylink-debian.git
cd displaylink-debian/ && sudo ./displaylink-debian.sh
```

* Deleting this made it start working

```bash
sudo dkms install -m evdi -v 1.14.1
```

* Restart to pick up config changes

```bash
sudo shutdown -r now
```
