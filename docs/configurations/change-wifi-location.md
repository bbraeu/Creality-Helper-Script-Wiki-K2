# Change WiFi Location — K2

The K2 connects to your local WiFi network during initial setup. If you move the printer or change your network, you can reconnect directly from the touchscreen.

---

## Change WiFi from Touchscreen

- On the touchscreen go to **Settings → Network**
- Tap your current network to disconnect, or tap **+** to add a new network
- Select your network from the list and enter the password
- The printer reconnects and displays the new IP address

---

## Find Your Printer's IP Address

After connecting, the IP address is shown in **Settings → Network**.

You can also find it from your router's DHCP client list — look for the printer's hostname (`K2-XXXX` on the K2, `K2Pro-XXXX` / `K2Plus-XXXX` on the K2 Pro / K2 Plus, with the last 4 characters of the MAC address; helper option 33 shows the hostname).

---

## Set a Static IP

For a stable IP that does not change between reboots, configure a DHCP reservation on your router (recommended). Assign a permanent IP to the printer's MAC address — this requires no changes on the printer itself and survives factory resets.

---

## Reconnect After Factory Reset

After a factory reset the printer loses its WiFi credentials. Reconnect from the touchscreen:

**Settings → Network → select your network → enter password**

Root access must also be re-enabled after a factory reset. See [Enable Root Access](../firmwares/install-and-update-rooted-firmware-k2.md#enable-root-access).
