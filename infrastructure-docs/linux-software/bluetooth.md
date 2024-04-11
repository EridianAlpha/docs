# 🔵 Bluetooth

1. **Launch bluetoothctl**: Type `bluetoothctl` and press Enter. This will open the Bluetooth control prompt, indicated by `[bluetooth]#`.
2.  **Turn on Bluetooth**: To ensure Bluetooth is powered on, type:

    ```csharp
    power on
    ```
3.  **Make the Device Discoverable**: If you want to make your Raspberry Pi discoverable by other devices, type:

    ```csharp
    discoverable on
    ```
4.  **Scan for Devices**: To start scanning for nearby Bluetooth devices, type:

    ```csharp
    scan on
    ```

    You will see a list of available devices and their MAC addresses.
5.  **Pair a Device**: Once you find the device you want to pair with (for example, your Bluetooth mouse), pair it using its MAC address:

    ```arduino
    pair [DEVICE MAC ADDRESS]
    ```

    Replace `[DEVICE MAC ADDRESS]` with the actual address of the device.
6.  **Connect to the Device**: After pairing, you’ll need to connect to the device:

    ```arduino
    connect [DEVICE MAC ADDRESS]
    ```
7.  **Trust the Device (Optional)**: To automatically connect to the device in the future, you might want to trust it:

    ```css
    trust [DEVICE MAC ADDRESS]
    ```
8.  **Stop Scanning**: Once you're done, you can turn off scanning:

    ```vbnet
    scan off
    ```
9. **Exit bluetoothctl**: Type `exit` to leave the bluetoothctl prompt.

Additional Commands:

*   **Show Devices**: To list all known devices, use:

    ```
    devices
    ```
*   **Show Paired Devices**: To list paired devices, use:

    ```
    paired-devices
    ```
*   **Remove a Device**: To remove/unpair a device, use:

    ```arduino
    remove [DEVICE MAC ADDRESS]
    ```
*   **Show Info**: For information about a specific device, use:

    ```css
    info [DEVICE MAC ADDRESS]
    ```

This should cover most of the basic usage scenarios for `bluetoothctl`. Remember, for specific devices (like some headphones or speakers), additional steps might be required for full functionality.
