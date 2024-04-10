# 🛜 ZeroTier

{% code title="Install ZeroTier" %}
```bash
sudo apt-get install curl -y
curl -s https://install.zerotier.com | sudo bash
```
{% endcode %}

{% code title="Join an existing network" %}
```bash
sudo zerotier-cli join <NETWORK_ID>
```
{% endcode %}
