### Automating Wi-Fi Setup On Windows 11 With A Rubber Ducky Script

#### Thanks to @bryce-kimber for some syntax and debug fixes

When you have to configure Wi-Fi on dozens of machines, clicking through the Windows GUI gets old fast. To speed things up, I built a Rubber Ducky script that automatically:

* Opens PowerShell
* Builds a valid Wi-Fi profile in XML
* Saves it to disk
* Imports it with `netsh`
* Connects the machine to the specified SSID

All the user has to do is plug in the Ducky. Below is a breakdown of how it works and the full script you can use or adapt.

---

## What This Script Does

On a Windows 11 machine, the script:

1. Launches PowerShell using the Run dialog.
2. Sets two variables:

   * `$ssid` for the Wi-Fi network name.
   * `$pass` for the Wi-Fi password.
3. Builds a Windows WLAN XML profile using those values.
4. Writes the XML profile to `C:\ProgramData\wifi-<SSID>.xml`.
5. Adds the Wi-Fi profile via `netsh wlan add profile`.
6. Connects to the SSID with `netsh wlan connect`.
7. Deletes the XML file for basic cleanup.
8. Exits PowerShell.

This is useful when you need to push a known SSID and password to many Windows 11 laptops quickly, and you are authorized to configure those devices.

---

## Requirements

* A USB Rubber Ducky or compatible HID injection device.
* Windows 11 machines, unlocked and logged into a user account that can modify Wi-Fi profiles.
* The target systems must already have Wi-Fi hardware and drivers installed.
* Keyboard layout must match what the script assumes (typically US layout). If not, you may need to adjust timings or key sequences.

Use this only on systems you own or are explicitly authorized to manage. This is an automation tool, not a way to bypass security.

---

## Script Walkthrough

### 1. Startup and PowerShell

The script begins by giving Windows time to recognize the Ducky as a keyboard and then opens PowerShell using the Run dialog:

* `GUI r` opens Run.
* Types `powershell` and presses Enter.
* Waits for PowerShell to launch.

### 2. SSID and Password Variables

Next, it sets two PowerShell variables:

```powershell
$ssid="YOUR_SSID_HERE"
$pass="YOUR_PASSWORD_HERE"
```

You should replace `YOUR_SSID_HERE` and `YOUR_PASSWORD_HERE` with your actual network name and password before flashing the payload to the Ducky.

### 3. Building the WLAN XML Profile

Windows stores Wi-Fi profiles as XML. The script uses a PowerShell here-string to assemble this XML:

```powershell
$xml=@"
<?xml version="1.0"?>
<WLANProfile xmlns="http://www.microsoft.com/networking/WLAN/profile/v1">
  <name>$ssid</name>
  <SSIDConfig>
    <SSID>
      <name>$ssid</name>
    </SSID>
  </SSIDConfig>
  <connectionType>ESS</connectionType>
  <connectionMode>auto</connectionMode>
  <MSM>
    <security>
      <authEncryption>
        <authentication>WPA2PSK</authentication>
        <encryption>AES</encryption>
        <useOneX>false</useOneX>
      </authEncryption>
      <sharedKey>
        <keyType>passPhrase</keyType>
        <protected>false</protected>
        <keyMaterial>$pass</keyMaterial>
      </sharedKey>
    </security>
  </MSM>
</WLANProfile>
"@
```

A key part here is the root element:

```xml
<WLANProfile xmlns="http://www.microsoft.com/networking/WLAN/profile/v1">
```

* `WLANProfile` is the root element for a wireless profile.
* The `xmlns` attribute defines the XML namespace that Windows expects for WLAN profiles. It tells Windows, “this XML follows the WLAN profile schema version 1,” which `netsh` and the OS can parse correctly.

The script hard-codes WPA2-PSK with AES:

* `<authentication>WPA2PSK</authentication>`
* `<encryption>AES</encryption>`

If you need WPA3 or enterprise authentication, you would need to adjust the XML accordingly.

### 4. Saving The Profile To Disk

The script writes the XML to a file in `ProgramData`:

```powershell
$path="$env:ProgramData\wifi-$ssid.xml"
$xml | Out-File -FilePath $path -Encoding ASCII -Force
```

Using `ProgramData` keeps things in a shared, non-user-specific location that is typically writable with standard privileges.

### 5. Adding The Profile And Connecting

Once the XML is written, the script calls `netsh`:

```powershell
netsh wlan add profile filename="$path"
netsh wlan connect name="$ssid"
```

* `netsh wlan add profile` imports the XML as a Wi-Fi profile.
* `netsh wlan connect` tells Windows to connect using that profile.

If the SSID is in range and the password is correct, the machine will associate to the network.

### 6. Cleanup

To avoid leaving the XML lying around, the script removes it:

```powershell
Remove-Item $path -Force
```

Then it exits PowerShell.

---

## Full Rubber Ducky Script

Here is the complete Ducky script payload as used:

```text
REM ==============================
REM Rubber Ducky script: Add WiFi
REM Windows 11, PowerShell + netsh
REM Replace YOUR_SSID_HERE and YOUR_PASSWORD_HERE
REM ==============================

DEFAULT_DELAY 20
DELAY 2000

GUI r
DELAY 500

STRING powershell
DELAY 300
ENTER

DELAY 1500

STRING $ssid="YOUR_SSID_HERE"
ENTER
DELAY 400

STRING $pass="YOUR_PASSWORD_HERE"
ENTER
DELAY 400

STRING $xml=@"
ENTER
DELAY 300

STRING <?xml version="1.0"?>
ENTER
STRING <WLANProfile xmlns="http://www.microsoft.com/networking/WLAN/profile/v1">
ENTER
STRING   <name>$ssid</name>
ENTER
STRING   <SSIDConfig>
ENTER
STRING     <SSID>
ENTER
STRING       <name>$ssid</name>
ENTER
STRING     </SSID>
ENTER
STRING   </SSIDConfig>
ENTER
STRING   <connectionType>ESS</connectionType>
ENTER
STRING   <connectionMode>auto</connectionMode>
ENTER
STRING   <MSM>
ENTER
STRING     <security>
ENTER
STRING       <authEncryption>
ENTER
STRING         <authentication>WPA2PSK</authentication>
ENTER
STRING         <encryption>AES</encryption>
ENTER
STRING         <useOneX>false</useOneX>
ENTER
STRING       </authEncryption>
ENTER
STRING       <sharedKey>
ENTER
STRING         <keyType>passPhrase</keyType>
ENTER
STRING         <protected>false</protected>
ENTER
STRING         <keyMaterial>$pass</keyMaterial>
ENTER
STRING       </sharedKey>
ENTER
STRING     </security>
ENTER
STRING   </MSM>
ENTER
STRING </WLANProfile>
ENTER

STRING "@
ENTER
DELAY 400

STRING $path="$env:ProgramData\wifi-$ssid.xml"
ENTER
DELAY 400

STRING $xml | Out-File -FilePath $path -Encoding ASCII -Force
ENTER
DELAY 800

STRING netsh wlan add profile filename="$path"
ENTER
DELAY 1200

STRING netsh wlan connect name="$ssid"
ENTER
DELAY 1500

STRING Remove-Item $path -Force
ENTER
DELAY 600

STRING exit
ENTER
```

Before you flash this to your Rubber Ducky:

* Replace `YOUR_SSID_HERE` with your actual Wi-Fi network name.
* Replace `YOUR_PASSWORD_HERE` with your actual Wi-Fi password.
* Adjust `DELAY` values if the target machines are slow to open PowerShell.

---

## Practical Notes

* This approach is ideal for environments like classrooms, labs, or offices where you control the endpoints and need a quick way to standardize Wi-Fi settings.
* If UAC prompts appear when launching PowerShell, the script may need to be adapted to run PowerShell without elevation or to handle the UAC dialog.
* For more complex environments (WPA2-Enterprise, certificate-based auth, etc.), the XML portion would need to match your exact configuration.
