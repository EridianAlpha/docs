---
icon: screwdriver
---

# Audit Tools

## Count Lines of Code (cloc)

{% embed url="https://github.com/AlDanial/cloc?tab=readme-ov-file#install-via-package-manager" %}

```bash
 cloc ./src
 
 # OUTPUT
      16 text files.
      16 unique files.
       0 files ignored.

github.com/AlDanial/cloc v 2.00  T=0.02 s (917.5 files/s, 144913.9 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Solidity                        16            335            837           1355
-------------------------------------------------------------------------------
SUM:                            16            335            837           1355
-------------------------------------------------------------------------------
```

## VCS Solidity Metrics Report&#x20;

{% embed url="https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-metrics" %}

## Slither

{% embed url="https://github.com/crytic/slither" %}

Use Docker to avoid Python dependency issues.

### Installation

{% code title="Pull latest image" %}
```bash
docker pull trailofbits/eth-security-toolbox
```
{% endcode %}

{% code title="Run and enter container" %}
```bash
docker run -it -v <PATH_TO_PROJECT_ROOT>:/share trailofbits/eth-security-toolbox
```
{% endcode %}

### Usage

{% code title="Slither commands inside container" %}
```bash
slither .                 # Run on all files
slither src/AavePM.sol    # Run on specific files
```
{% endcode %}

## Aderyn

{% embed url="https://github.com/Cyfrin/aderyn" %}

### Installation

```bash
curl -L https://raw.githubusercontent.com/Cyfrin/aderyn/dev/cyfrinup/install | bash
```

```bash
cyfrinup
```

### Usage

```bash
aderyn .
```
