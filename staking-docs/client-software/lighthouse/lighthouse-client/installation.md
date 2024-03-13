---
description: Lighthouse client installation guide.
---

# 💾 Installation

* [Create Aliases](installation.md#create-aliases)
* [Rust - Install](installation.md#rust-install)
* [Lighthouse - Install](installation.md#lighthouse-install)
* [Lighthouse - Update Scripts](installation.md#lighthouse-update-scripts)

### Create Aliases

```bash
echo "alias lighthouse-version-current='/usr/local/bin/lighthouse --version'" >> ~/.bashrc
echo "alias lighthouse-build='~/lighthouse-build.sh'" >> ~/.bashrc
echo "alias lighthouse-version-new='~/.cargo/bin/lighthouse --version'" >> ~/.bashrc
echo "alias lighthouse-deploy='~/lighthouse-deploy.sh'" >> ~/.bashrc

source ~/.bashrc
```

### Rust - Install

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

<1>
<ENTER>

source $HOME/.cargo/env
```

Make sure `protoc` is installed otherwise the build will fail.

```bash
sudo apt-get install -y protobuf-compiler
protoc --version
```

### Lighthouse - Install

Build the latest version of `Lighthouse`.

```bash
LIGHTHOUSE_VERSION_COMMIT_HASH=        # e.g.441fc16 

cd ~
git clone -b stable https://github.com/sigp/lighthouse.git
cd ~/lighthouse
git checkout ${LIGHTHOUSE_VERSION_COMMIT_HASH}
make
```

Move the compiled `Lighthouse` build to a new directory.

```bash
sudo cp ~/.cargo/bin/lighthouse /usr/local/bin
```

Check version.

```bash
/usr/local/bin/lighthouse --version
```

Create `Lighthouse` directory.

```bash
sudo mkdir -p /var/lib/lighthouse
```

### Lighthouse - Update Scripts

Create `Lighthouse` build script.

```bash
vim ~/lighthouse-build.sh
```

{% code title="~/lighthouse-build.sh" %}
```bash
#!/bin/bash
set -e
cd ~/lighthouse

read -p "Enter the commit hash you want to checkout: " commit_hash

git fetch
git checkout $commit_hash

echo
echo "********************"
echo "Making Lighthouse..."
echo "********************"
make
```
{% endcode %}

Create `Lighthouse` deploy script.

```bash
vim ~/lighthouse-deploy.sh
```

{% code title="~/lighthouse-deploy.sh" %}
```bash
#!/bin/bash
# set -e     # This isn't working correctly, so comment out for now

while true; do
    read -p "Are you sure you want to deploy Lighthouse? (Y/N) " yn
    case $yn in
        [Yy]* ) break;;
        [Nn]* ) exit;;
        * ) echo "Please answer Y or N.";;
    esac
done

# Check if the services exist before checking their status
if systemctl list-units --full -all | grep -Fq lighthousebeacon.service; then
    beacon_status=$(sudo systemctl is-active lighthousebeacon.service)
else
    beacon_status="not_installed"
fi

if systemctl list-units --full -all | grep -Fq lighthousevalidator.service; then
    validator_status=$(sudo systemctl is-active lighthousevalidator.service)
else
    validator_status="not_installed"
fi

echo "**********************"

if [ "$beacon_status" = "active" ]; then
    echo "Stopping Lighthouse Beacon..."
    sudo systemctl stop lighthousebeacon.service
elif [ "$beacon_status" = "not_installed" ]; then
    echo "Warning: Lighthouse Beacon service is not installed."
fi

if [ "$validator_status" = "active" ]; then
    echo "Stopping Lighthouse Validator..."
    sudo systemctl stop lighthousevalidator.service
elif [ "$validator_status" = "not_installed" ]; then
    echo "Warning: Lighthouse Validator service is not installed."
fi

echo "Replacing previous version..."

# Check if the file exists before trying to remove it
if [ -f /usr/local/bin/lighthouse ]; then
    sudo rm /usr/local/bin/lighthouse
fi

# Check if the source file exists before copying
if [ -f ~/.cargo/bin/lighthouse ]; then
    sudo cp ~/.cargo/bin/lighthouse /usr/local/bin
else
    echo "Error: Source file does not exist. Installation aborted."
    exit 1
fi

if [ "$beacon_status" = "active" ]; then
    echo "Restarting Lighthouse Beacon..."
    sudo systemctl start lighthousebeacon.service
fi

if [ "$validator_status" = "active" ]; then
    echo "Restarting Lighthouse Validator..."
    sudo systemctl start lighthousevalidator.service
fi
```
{% endcode %}

Make all scripts executable.

```bash
chmod u+x ~/lighthouse-build.sh
chmod u+x ~/lighthouse-deploy.sh
```
