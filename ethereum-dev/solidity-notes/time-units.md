# Time Units

Solidity provides some native units for dealing with time.

The variable `now`will return the current unix timestamp of the latest block (the number of seconds that have passed since January 1st 1970). The unix time as I write this is `1515527488`.

{% hint style="info" %}
Unix time is traditionally stored in a 32-bit number. This will lead to the "Year 2038" problem, when 32-bit unix timestamps will overflow and break a lot of legacy systems.

So if we wanted our DApp to keep running 20 years from now, we could use a 64-bit number instead — but our users would have to spend more gas to use our DApp in the meantime. Design decisions!
{% endhint %}

Solidity also contains the time units

* `seconds`
* `minutes`
* `hours`
* `days`
* `weeks`
* `years`

These will convert to a uint of the number of seconds in that length of time.

Here's an example of how these time units can be useful:

```solidity
uint lastUpdated;

// Set `lastUpdated` to `now`
function updateTimestamp() public {
  lastUpdated = now;
}

// Will return `true` if 5 minutes have passed since `updateTimestamp` was 
// called, `false` if 5 minutes have not passed
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

```solidity
uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime))) - 1;
```

{% hint style="info" %}
The `uint32(...)` is necessary because `now`returns a `uint256`by default. So we need to explicitly convert it to a `uint32`.
{% endhint %}
