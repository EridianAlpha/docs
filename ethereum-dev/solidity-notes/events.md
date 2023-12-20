# Events

Events are a way for your contract to communicate that something happened on the blockchain to your app front-end, which can be 'listening' for certain events and taking action when they happen.

{% hint style="info" %}
It's best practice to have events emitted before the "Interactions" section of your function (in the [CEI model](cei-checks-effects-interactions.md)).
{% endhint %}

* You can have up to 3 `indexed parameters` (also know as `topics`) per individual event.
* Event logs can't be accessed by smart contracts, so are only used for off-chain applications such as web apps.

```solidity
contract SimpleStorage {
    uint256 favoriteNumber;
    event storedNumber(
        uint256 indexed oldNumber,
        uint256 indexed newNumber,
        uint256 addedNumber,
        address sender
    );

    function store(uint256 _favoriteNumber) public {
        emit storedNumber(
            favoriteNumber,
            _favoriteNumber,
            _favoriteNumber + favoriteNumber,
            msg.sender
        );
        favoriteNumber = _favoriteNumber;
    }

    function retrieve() public view returns (uint256) {
        return favoriteNumber;
    }
}
```
