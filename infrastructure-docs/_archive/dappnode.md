---
description: Installing Dappnode on a vanilla Ubuntu server machine.
---

# 📱 Dappnode

### Install Dappnode

* [https://docs.dappnode.io/user/quick-start/core/installation/](https://docs.dappnode.io/user/quick-start/core/installation/)

{% code title="Dappnode Installation Scripts" %}
```bash
sudo wget -O - https://prerequisites.dappnode.io | sudo bash
sudo wget -O - https://installer.dappnode.io | sudo bash
shutdown -r now
```
{% endcode %}

### Access Dappnode

* [http://my.dappnode](http://my.dappnode)
* [http://dappnode.local](http://dappnode.local)

### Config

* System -> Advanced -> Change Dappnode name
  * Name the Dappnode!
* Turn WiFi `OFF`
* System -> Auto updates
  * System packages `Enable`
  * My packages `Disable`
    * This is to stop the custom network configuration from being deleted on each package when it's updated.
    * This is a known bug that I've talked to Dappnode about, but as of writing this, it's not fixed.







