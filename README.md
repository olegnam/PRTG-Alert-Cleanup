# PRTG Decommission Alert Cleanup
 
## Background
 
I needed to resolve a few hundred active alerts tied to devices that had been decommissioned. Previous administrators had left notes such as "decomm" in the sensor messages instead of removing the devices. PRTG does not offer a native way to search sensor messages and bulk remove the parent devices, so cleaning this up by hand would have meant clicking through hundreds of objects one at a time.
 
To solve this, I used the PRTG HTTP API with an API key to pull every sensor in a single request, filter on decommission keywords, and produce a deduplicated list of affected devices. 
 
## What the Script Does
 
The script queries the PRTG API sensors, checks each sensor message against a list of keywords, and records the parent device of every match. Because one device can have many sensors carrying the same note, the script deduplicates by device ID so each device appears only once. Results are printed to the console and exported to a CSV file for review.
 
The script is read only. It does not delete or modify anything in PRTG.
 
## Requirements
 
You will need Python 3.8 or newer, the `requests` library, network access to your PRTG server, and a PRTG user account that is allowed to create API keys.
 
```bash
pip install -r requirements.txt
```
 
## Step 1: Obtain Your PRTG URL
 
Log in to the PRTG web interface in your browser. Copy the scheme and host from the address bar, for example `https://10.0.0.x` or `https://prtg.example.com`. Do not include any path after the host. Paste this value into the `PRTG_URL` variable in `find_decomm_devices.py`.
 
## Step 2: Create an API Key
 
In the PRTG web interface, go to **Setup > Account Settings > API Keys**. You can also open this page directly at `https://<yourprtgserver>/myaccount.htm?tabid=5`.
 
Hover over the plus icon and select **Add API Key**. Give the key a name and description that explain its purpose. Leave the key type as **Scripting**, which works with the PRTG API. For this script, **Read access** is sufficient because it only reads sensor data.
 
Copy the API key before clicking **OK**. PRTG will not show the key again. If you lose it, delete the key and create a new one.
 
Reference: [PRTG Manual: API Keys](https://www.paessler.com/manuals/prtg/api_keys)
 
## Step 3: Store the API Key as an Environment Variable
 
The script reads the key from an environment variable so it is never written into the code or committed to source control. The CSV provides a reviewable list to delete at the device level which clears all related alerts at once.
 
Windows PowerShell:
 
```powershell
$env:PRTG_API_KEY = "your_api_key_here"
```
 
macOS or Linux:
 
```bash
export PRTG_API_KEY="your_api_key_here"
```
 
## Step 4: Customize and Run
 
Update the `KEYWORDS` list to match the notes used in your environment. Matching is not case sensitive. Then run:
 
```bash
python find_decomm_devices.py
```
 
The script prints every matching sensor, the total number of unique devices found, and the location of the exported CSV.
 
## Example Output
 
```
Fetching sensors...
Match found - Device: OLD-SRV-01 | Device ID: 2045 | Message: Decomm per change ticket
Match found - Device: OLD-SRV-01 | Device ID: 2045 | Message: decomm
Match found - Device: LEGACY-SW-03 | Device ID: 3112 | Message: Redacted Decomm
 
Total devices that would be deleted: 2
Results exported to: decomm_devices.csv
```
 
## Security Notes
 
SSL certificate verification is disabled because many PRTG servers use self-signed certificates. If your server has a trusted certificate, change `verify=False` to `verify=True`. Exported CSV files are excluded from the repository through `.gitignore` because they contain internal device names. Delete the API key in PRTG when the cleanup is complete.
 
