# ☁️ Ethers

How to use a custom struct in Ethers

```solidity
struct UpgradeHistory {
    string version;
    uint256 upgradeTime;
    address upgradeInitiator;
}
```

{% code fullWidth="true" %}
```javascript
const aavePMAbi = [
    "function getUpgradeHistory() external view returns (tuple(string version, uint256 upgradeTime, address upgradeInitiator)[] memory)",
    ]
    
const upgradeHistoryData = await aavePM.getUpgradeHistory()
aavePMData.upgradeHistory = upgradeHistoryData.map((item) => ({
    version: item.version,
    upgradeTime: BigNumber(item.upgradeTime).toNumber(),
    upgradeInitiator: item.upgradeInitiator,
}))


{aavePMData?.upgradeHistory
    ?.sort((b, a) => a.upgradeTime - b.upgradeTime)
    .map((upgrade, index) => (
        <UpgradeDisplay
            key={index}
            version={upgrade.version}
            upgradeTime={upgrade.upgradeTime}
            upgradeInitiator={upgrade.upgradeInitiator}
        />
))}
```
{% endcode %}





