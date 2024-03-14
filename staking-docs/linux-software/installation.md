---
description: >-
  Linux software installation guide, starting with a new machine through to the
  point of installing the validator clients.
---

# 💾 Installation

### 💾 Installing Linux

To avoid duplication these details can be found on the EthStaker Knowledge Base.

{% content-ref url="https://app.gitbook.com/s/KnJhWg57YoZq2MPfatKE/tutorials/installing-linux" %}
[Installing Linux](https://app.gitbook.com/s/KnJhWg57YoZq2MPfatKE/tutorials/installing-linux)
{% endcontent-ref %}

### 🔧 System Configuration

Add useful `Bash` aliases.

{% code fullWidth="true" %}
```bash
echo "alias lsl='ls -la'" >> ~/.bashrc
echo "alias clc='clear'" >> ~/.bashrc
echo "alias myip='echo Response from https://ipinfo.io/ip:; curl https://ipinfo.io/ip; echo'" >> ~/.bashrc
echo "alias ports='sudo lsof -i -P -n | grep LISTEN'" >> ~/.bashrc
echo "alias update-system='sudo apt-get update -y; sudo apt-get dist-upgrade -y'" >> ~/.bashrc
echo "alias update-firmware='fwupdmgr refresh; fwupdmgr get-updates; fwupdmgr update'" >> ~/.bashrc
echo "alias update-kernel='sudo apt-get install -y linux-image-generic-hwe-22.04'" >> ~/.bashrc
echo "alias daemon-reload='sudo systemctl daemon-reload'" >> ~/.bashrc
source ~/.bashrc
```
{% endcode %}

Update system packages.

```bash
update-system
```

Update kernel to the latest version for 22.04.

```bash
update-kernel
```

Install useful packages.

```sh
sudo apt-get install -y \
git \
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
fwupd \
linux-tools-common \
linux-tools-generic
```

```bash
sudo snap install btop
```

Update [firmware](https://github.com/fwupd/fwupd).

```bash
update-firmware
```

Restart machine.

```bash
sudo shutdown -r now
```

Update the system again after restarting.

```bash
update-system
```

Set up `btop`.

```bash
btop
```

* Press `ESC` key to see menu and select `OPTIONS`.
* Change theme to TTY.
* Change time interval to 1000ms.

### 🚪 Change Default SSH Port

Useful when you have multiple machines running on the same ip address.

```bash
sudo vim /etc/ssh/sshd_config
```

Uncomment `#Port 22` and change it to your modified ssh port.

Restart the `sshd` service.

```bash
sudo systemctl restart sshd
```

### 🚧 Firewall Configuration

Confirm UFW is disabled before blocking everything or you could be disconnected.

```bash
sudo ufw disable
sudo ufw status
```

Configure the firewall.

```bash
CUSTOM_SSH_PORT=                  # Default: 22
MOSH_STARTING_PORT=               # Default: 60000
MOSH_ENDING_PORT=                 # Default: 61000

sudo ufw default deny incoming comment 'Deny all incoming traffic'
sudo ufw default allow outgoing comment 'Allow all outgoing traffic'

sudo ufw allow ${CUSTOM_SSH_PORT} comment 'Allow custom ssh in'
sudo ufw allow ${MOSH_STARTING_PORT}:${MOSH_ENDING_PORT}/udp comment 'Allow Mosh in'
```

Edit the UFW settings to disable IPV6. This isn't for any particular security reason, it just makes the UFW status easier to read when it prints out on screen.

```bash
sudo vim /etc/default/ufw
```

Change `IPV6` to `no`.

```bash
IPV6=no
```

Enable UFW.

```bash
# <USE --FORCE TO STOP THE PROMPT ASKING IF YOU WANT TO ENABLE>
sudo ufw --force enable

sudo ufw status verbose
```

### 📅 noatime

By default, Linux will write a new file timestamp on every read. As you may imagine, this is no bueno for database applications like an Ethereum node.

You can increase the lifetime of your SSD - and incidentally get a small speed boost - by turning this "atime" feature off. [\[1\]](https://eth-docker.net/Usage/LinuxSecurity#noatime)

```bash
sudo vim /etc/fstab
```

Find the entry for your `/` filesystem.

{% code title="/etc/fstab" fullWidth="false" %}
```bash
# BEFORE
# /dev/disk/by-id/dm-uuid-LVM-u1...i3uPy21O / ext4 defaults 0 1

# AFTER
/dev/disk/by-id/dm-uuid-LVM-u1...i3uPy21 / ext4 defaults,noatime 0 1
```
{% endcode %}

Don't delete any parameters, just add `,noatime`. And make sure to add that to the 4th column, not anywhere else.

Save the file, and if the following command then runs without any errors, the configuration is correct.

```
sudo mount -o remount /
```

### 🔁 Turn off swap

If you have a lot of RAM you can turn off swap completely. [\[2\]](https://eth-docker.net/Usage/LinuxSecurity#swappiness)

```
sudo swapoff -a
```

```bash
sudo vim /etc/fstab
```

Find the entry for your `/` filesystem.

{% code title="/etc/fstab" fullWidth="false" %}
```bash
# BEFORE
/swapfile swap swap defaults 0 0

# AFTER
#/swapfile swap swap defaults 0 0
```
{% endcode %}

### 🎚️ cpupower

Some CPUs have a power saving feature that reduces their speed when not in use. This doesn't work well for Ethereum nodes as the time between blocks can cause CPUs to slow down... only to need them to speed back up again when a block arrives.

```bash
cpupower

# <INSTALL THE VERSION IT THEN SUGGESTS>
sudo apt-get install -y <SUGGESTED_VERSION>

cpupower frequency-info

# <CHECK THAT THIS ROW EXISTS>
# available cpufreq governors: performance powersave
```

If the performance option exists, set that.

```bash
sudo cpupower frequency-set -g performance
```

Then create a service to set `performance` after reboot as that setting doesn't persist on its own.

```bash
sudo vim /etc/systemd/system/cpupower.service
```

{% code title="/etc/systemd/system/cpupower.service" %}
```ini
[Unit]
Description=Set CPU governor to performance

[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower frequency-set -g performance

[Install]
WantedBy=multi-user.target
```
{% endcode %}

```bash
sudo systemctl daemon-reload
sudo systemctl enable cpupower.service
sudo systemctl status cpupower.service
```

### 📏 Use all Available Disk Space

```bash
sudo lvdisplay # Check your logical volume size

sudo lvm # Attach the lvm console
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
exit

sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
df -h # Check results
```

### 🫣 Hide Welcome Message on Login

Some of the messages are useful so I don't want to hide everything, but some are annoying.

Hide parts of the message by making the template `non-executable`.

```bash
sudo chmod -x /etc/update-motd.d/10-help-text
sudo chmod -x /etc/update-motd.d/50-motd-news
```

### ⏱️ Increases Service Shutdown Timer

I was concerned that the default time of 180 seconds wouldn't be long enough for all the services running to gracefully shutdown.

The shutdown time can be individually set on each service, but this is a catch-all.

```bash
sudo vim /etc/systemd/system.conf
```

Uncomment out the line `#DefaultTimeoutStopSec=90s` and change it to `1200s`.

### 🛑 Brute-Force SSH Protection

To protect your server from brute-force SSH connection attempts, you can install `fail2ban` which will monitor incoming connections and block IP addresses that try to log in with faulty credentials repeatedly.

```bash
sudo apt-get install -y fail2ban
sudo vim /etc/fail2ban/jail.d/ssh.local
```

{% hint style="warning" %}
If you're using a non-standard SSH port that isn't `22` then you will need to change that in the cofig file below.
{% endhint %}

{% code title="/etc/fail2ban/jail.d/ssh.local" %}
```bash
[sshd]
enabled = true
banaction = ufw
port = <SSH_PORT>
filter = sshd
logpath = %(sshd_log)s
maxretry = 5
```
{% endcode %}

You can change the `maxretry` setting, which is the number of attempts it will allow before locking the offending address out.

Save the file and restart the service.

```bash
sudo systemctl restart fail2ban
```

### 📱 SSH Security - 2FA

Install the Google Authenticator module on your node with this command:

```bash
sudo apt-get install -y libpam-google-authenticator
```

Now tell the `PAM` (pluggable authentication modules) to use this module.

```bash
sudo vim /etc/pam.d/sshd
```

Find `@include common-auth` (it should be at the top) and comment it out by adding a `#` in front of it, so it looks like this:

```
# Standard Un*x authentication.
#@include common-auth
```

Next, add these lines to the top of the file:

```
# Enable Google Authenticator
auth required pam_google_authenticator.so
```

Then save and exit.

Now that `PAM` knows to use Google Authenticator, the next step is to tell `sshd` to use `PAM`.&#x20;

```bash
sudo vim /etc/ssh/sshd_config
```

Now change the line `KbdInteractiveAuthentication no` to `KbdInteractiveAuthentication yes` so it looks like this:

```
# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication yes
```

(Older versions of SSH call this option `ChallengeResponseAuthentication` instead of `KbdInteractiveAuthentication`.)

Add the following line to the bottom of the file, which indicates to `sshd` that it needs both an SSH key and the Google Authenticator code:

```
AuthenticationMethods publickey,keyboard-interactive
```

Every option added to `AuthenticationMethods` will be required when you log in. So you can choose e.g. 2FA and password, or a combination of all three methods.

* `publickey` (SSH key)
* `password publickey` (password)
* `keyboard-interactive` (2FA verification code)

Then save and exit.

Now that `sshd` is set up, we need to create our 2FA codes. In your terminal, run:

```
google-authenticator
```

First, it will ask you about time-based tokens. Say `y` to this question:

```
Do you want authentication tokens to be time-based: y
```

You will now see a big QR code on your screen; scan it with your Google Authenticator app to add it. You will also see your secret and a few backup codes looking like this:

```
Your new secret key is: IRG2TALMR5U2LK5VQ5AQIG3HA4
Your verification code is 282436
Your emergency scratch codes are:
  29778030
  86888537
  50553659
  41403052
  82649596
```

{% hint style="info" %}
Record the emergency scratch codes somewhere safe in case you need to log into the machine but don't have your 2FA app handy. Without the app, you will no longer be able to SSH into the machine!
{% endhint %}

Finally, it will ask you for some more parameters; the recommended defaults are as follows:

```
Do you want me to update your "/<username>/.google_authenticator" file: y
Do you want to disallow multiple uses of the same authentication token: y
By default... < long story about time skew > ... Do you want to do so: n
Do you want to enable rate-limiting: y
```

Once you're done, restart `sshd` so it grabs the new settings:

```bash
sudo systemctl restart sshd
```

When you try to SSH into your server with your SSH keys, you should now also be asked for a 2FA verification code, but not for a password.

### 🐳 Install Docker

{% code fullWidth="true" %}
```bash
# Update the package index
sudo apt-get update -y

# Install packages to allow apt to use a repository over HTTPS
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release
    
# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up the stable repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package index again
sudo apt-get update -y

# Install the latest version of Docker Engine and containerd
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# To use Docker without sudo, add your current user to the docker group
sudo usermod -aG docker $USER
```
{% endcode %}

Log out and log back in or restart your machine for the group membership to take effect. You should now be able to run Docker commands without `sudo`.

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

### 📝 Systemd Journal Logs

The systemd services create logs that grow over time. It is possible to clear the logs to free up disk space on your server. The following steps will delete existing log data.

Be careful if you require this data for debugging purposes.

Check the amount of disk space the logs are using.

```bash
sudo journalctl --disk-usage
```

To clear the logs use the following command.

* The `--flush` flag flushes the logs currently in memory onto the disk.
* The `--rotate` flag archives the existing logs so they can’t be written to anymore and starts new logs for each service.
* The `--vacuum-time` flag deletes log data that is older than `28 days`.

```bash
sudo journalctl --flush --rotate
sudo journalctl --vacuum-time=28days
```

It is recommended to check the logs are in a good state after the vacuum operation.

Each log should have a status of `PASS`.

```bash
sudo journalctl --verify
```

To have the system automatically keep log data to a specified max size complete the following additional steps.

Open the systemd journal service configuration file.

```bash
sudo vim /etc/systemd/journald.conf
```

Edit the file to set the maximum disk space that can be used by the journal in persistent storage.

Remove the `#` from the line `SystemMaxUse` and add a value in megabytes, say `10000M`.

Restart the `journald` after updating the file.

```bash
sudo systemctl restart systemd-journald
```

Journal logs will now be limited to 1000MB in size.

***

Success! The staking machine is now ready to install the [client software](../client-software/) 🥳
