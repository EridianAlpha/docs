# Ubuntu Desktop

* Install Ubuntu Desktop

### 🔧 System Configuration

Add useful `Bash` aliases.

```bash
echo "alias lsl='ls -la'" >> ~/.bashrc
echo "alias clc='clear'" >> ~/.bashrc
echo "alias myip='echo Response from https://ipinfo.io/ip:; curl https://ipinfo.io/ip; echo'" >> ~/.bashrc
echo "alias ports='sudo lsof -i -P -n | grep LISTEN'" >> ~/.bashrc
echo "alias update-system='sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt-get dist-upgrade -y'" >> ~/.bashrc
echo "alias update-firmware='fwupdmgr refresh; fwupdmgr get-updates; fwupdmgr update'" >> ~/.bashrc
echo "alias daemon-reload='sudo systemctl daemon-reload'" >> ~/.bashrc
source ~/.bashrc
```

Update system packages.

{% tabs %}
{% tab title="Command Alias" %}
```bash
update-system    # Update all system packages
```
{% endtab %}

{% tab title="Full Commands" %}
```bash
sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt-get dist-upgrade -y    # Update all system packages
```
{% endtab %}
{% endtabs %}

Install useful packages.

<pre class="language-sh"><code class="lang-sh"><strong>sudo apt-get install -y \
</strong>git \
build-essential \
software-properties-common \
pkg-config \
cmake \
clang \
wget \
curl \
ccze \
vim \
mlocate \
jq \
net-tools \
unzip \
screen \
mosh \
ufw \
fwupd
</code></pre>

```bash
sudo snap install btop
```

Update [firmware](https://github.com/fwupd/fwupd).

{% tabs %}
{% tab title="Command Alias" %}
```bash
update-firmware  # Update firmware
```
{% endtab %}

{% tab title="Full Commands" %}
```bash
fwupdmgr refresh; fwupdmgr get-updates; fwupdmgr update     # Update firmware
```
{% endtab %}
{% endtabs %}

Set up `btop`.

```bash
btop
```

* Press `ESC` key to see menu and select `OPTIONS`.
* Change theme to TTY.
* Change time interval to 1000ms.

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

* Change keyboard shortcut for `Full Screen` to `Ctrl + Enter`
  * Hamburger button in terminal -> Preferences -> Shortcuts -> `Full Screen`
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

### 🚦 Git Configuration

Edit the Git configuration.

```bash
vim ~/.gitconfig
```

Add this to the `[alias]` section for a pretty view of commit tree for current branch.

```
[alias] tree = log --color --graph --pretty=format:'%Cred%h%Creset -%C(bold magenta)%d%Creset %s %C(cyan)(%cr)%C(bold blue) <%an> %Creset' --date=relative --abbrev-commit --all
[alias] tree-current = log --color --graph --pretty=format:'%Cred%h%Creset -%C(bold magenta)%d%Creset %s %C(cyan)(%cr)%C(bold blue) <%an> %Creset' --date=relative --abbrev-commit --all
```

Then run these commands in a directory using Git to see a pretty view of the commit tree.

```bash
git tree
git tree-current
```



