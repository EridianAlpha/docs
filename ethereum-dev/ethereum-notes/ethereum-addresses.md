# 🪧 Ethereum Addresses

## Ethereum Addresses

Ethereum addresses are 40 hexadecimal characters long, excluding the "0x" prefix. This makes each address 20 bytes (160 bits) in length.

* **20 Bytes Long**: An Ethereum address is derived from the last 20 bytes of the Keccak-256 hash of the public key.
* **160 Bits**: This length provides a vast address space, which helps avoid address collisions and enhances security.

### Step-by-Step Example

{% embed url="https://codepen.io/EridianAlpha/pen/NWVvBaQ" fullWidth="true" %}
Live example showing Ethereum address derivation from a private key
{% endembed %}

<details>

<summary>Code to generate an Ethereum address from a private key</summary>

{% code fullWidth="true" %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Ethereum Public Key Generator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.13.0/ethers.umd.min.js"></script>
    <style>
        .inline-label {
            font-weight: bold;
            display: inline;
        }
        .value {
            margin-top: 0px;
        }
        .highlight {
            color: cornflowerblue;
        }
    </style>
</head>
<body style="background-color: #14171C; color: white;">
    <label class="inline-label" for="privateKey">1. Enter Private Key:</label>
    <input  style="width: 500px;" type="text" id="privateKey" placeholder="0x..." value="0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80">
    <button onclick="generatePublicKey()">Generate Public Key</button>
    <br>
    <br>
    <pre class="inline-label">2. Public Key:</pre>
    <pre class="value" id="publicKey">...</pre>
    <pre class="inline-label">3. Keccak-256 Hash of Public Key:</pre>
    <pre class="value" id="publicKeyHash">...</pre>
    <pre class="inline-label">4. Ethereum Address:</pre>
    <pre class="value" id="ethereumAddress">...</pre>

    <script>
        let wallet;
        let publicKey;
        function generatePublicKey() {
            const privateKeyInput = document.getElementById('privateKey').value.trim();

            // Ensure the private key starts with "0x"
            const privateKey = privateKeyInput.startsWith('0x') ? privateKeyInput : '0x' + privateKeyInput;

            // Validate the private key length (64 characters for the hex representation, 66 with "0x")
            if (privateKey.length !== 66) {
                document.getElementById('publicKey').innerText = 'Invalid private key length';
                return;
            }

            try {
                // Create the Ethers wallet object from the privateKey
                wallet = new ethers.Wallet(privateKey);
                publicKey = wallet.signingKey.publicKey;
                document.getElementById('publicKey').innerText = publicKey;
            } catch (error) {
                document.getElementById('publicKey').innerText = 'Invalid publicKey';
                wallet = null;
                publicKey = null;
                console.error(error);
            }

            try {
                // Slice the `0x04` from the start of the publicKey
                const publicKeyFormatted = '0x' + publicKey.slice(4);

                // Convert the public key to a byte array using ethers.getBytes.
                const publicKeyBytes = ethers.getBytes(publicKeyFormatted);
                
                // Perform Keccak-256 hash of the public key
                const publicKeyHash = ethers.keccak256(publicKeyBytes);

                // Highlight the last 40 characters
                const start = publicKeyHash.slice(0, -40);
                const end = publicKeyHash.slice(-40);
                const highlightedHash = `${start}<span class="highlight">${end}</span>`;
                document.getElementById('publicKeyHash').innerHTML = highlightedHash;
            } catch (error) {
                document.getElementById('publicKeyHash').innerText = 'Invalid publicKeyHash';
                console.error(error);
            }

            try {
                const ethereumAddress = wallet.address

                // Highlight the last 40 characters
                const start = ethereumAddress.slice(0, -40);
                const end = ethereumAddress.slice(-40);
                const highlightedEthereumAddress = `${start}<span class="highlight">${end}</span>`;
                document.getElementById('ethereumAddress').innerHTML = highlightedEthereumAddress;
            } catch (error) {
                document.getElementById('ethereumAddress').innerText = 'Invalid ethereumAddress';
                console.error(error);
            }
        }
        window.onload = generatePublicKey;
    </script>
</body>
</html>

```
{% endcode %}

</details>

1. **Generate a Private Key.** This is a random 256-bit number. For simplicity, let's use a hexadecimal representation:&#x20;

{% code title="Private Key" %}
```
0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```
{% endcode %}

2. **Generate the Public Key.** The public key is derived from the private key using Elliptic Curve Cryptography (ECC), specifically the secp256k1 curve. The resulting public key is a 512-bit number (128 hexadecimal characters):

```javascript
wallet = new ethers.Wallet(privateKey);
publicKey = wallet.signingKey.publicKey;
```

{% code title="Public Key" overflow="wrap" %}
```
0x048318535b54105d4a7aae60c08fc45f9687181b4fdfc625bd1a753fa7397fed753547f11ca8696646f2f3acb08e31016afac23e630c5d11f59f61fef57b0d2aa5
```
{% endcode %}

3. **Keccak-256 Hash of the Public Key.** Ethereum uses the Keccak-256 hash function (a variant of SHA-3) to hash the public key. Note that only the x and y coordinates of the public key (excluding the initial '0x04' byte) are hashed.

```javascript
// Slice the `0x04` from the start of the publicKey
const publicKeyFormatted = '0x' + publicKey.slice(4);

// Convert the public key to a byte array using ethers.getBytes.
const publicKeyBytes = ethers.getBytes(publicKeyFormatted);

// Perform Keccak-256 hash of the public key
const publicKeyHash = ethers.keccak256(publicKeyBytes);
```

{% code title="Public Key (no '0x04')" overflow="wrap" %}
```
0x8318535b54105d4a7aae60c08fc45f9687181b4fdfc625bd1a753fa7397fed753547f11ca8696646f2f3acb08e31016afac23e630c5d11f59f61fef57b0d2aa5
```
{% endcode %}

{% code title="Keccak-256 Hash" %}
```
0xc1ffd3cfee2d9e5cd67643f8f39fd6e51aad88f6f4ce6ab8827279cfffb92266
```
{% endcode %}

{% hint style="info" %}
The initial '0x04' byte is a prefix used in the uncompressed format of an elliptic curve public key. This prefix indicates that the public key is represented in an uncompressed form, which includes both the x and y coordinates of the point on the elliptic curve.

**Uncompressed Format**:

* The public key consists of a prefix '0x04' followed by the x coordinate and the y coordinate of the elliptic curve point.
* The format is: `0x04 <x-coordinate> <y-coordinate>`

**Compressed Format**:

* The public key consists of a prefix '0x02' or '0x03' followed by only the x coordinate.
* The prefix '0x02' is used if the y coordinate is even, and '0x03' is used if the y coordinate is odd.

The '0x04' prefix is removed because it is not part of the essential elliptic curve point data (x and y coordinates). It merely indicates that the data following it is an uncompressed public key. For the purpose of hashing and deriving the Ethereum address, only the actual coordinates of the elliptic curve point are relevant, and thus the prefix is excluded.
{% endhint %}

4. **Ethereum Address.** The Ethereum address is derived from the last 20 bytes of the Keccak-256 hash.

```javascript
const ethereumAddress = wallet.address
```

{% code title="Ethereum Address" %}
```
0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
```
{% endcode %}
