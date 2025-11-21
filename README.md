# Azure Honeypot Lab Using Microsoft Sentinel (SIEM)
A complete end-to-end security project where a Windows honeypot VM is deployed in Microsoft Azure and integrated with Microsoft Sentinel to detect failed login attacks, enrich logs using GeoIP data, and visualize attacker locations on a real-time attack map.
This lab demonstrates practical SOC skills including log forwarding, KQL querying, SIEM configuration, Sentinel watchlists, and workbook creation.

## Project Overview

This project simulates a real-world cyberattack on a purposely exposed Windows Virtual Machine (honeypot). The VM logs all failed login attempts. These logs are forwarded to Log Analytics Workspace and visualized in Microsoft Sentinel.

The lab includes:

✔ Creating Azure resources

✔ Deploying a Windows VM honeypot

✔ Generating failed RDP login attempts

✔ Configuring Log Analytics & Sentinel

✔ Setting up Azure Monitoring Agent(AMA) + Data Collection Rukes(DCR) for log ingestion

✔ Enriching logs with GeoIP data using Sentinel Watchlists

✔ Creating an interactive attack map

✔ Writing & running KQL queries

## Part 1 — Azure Account Setup

### Step-1: Create a Free Azure Account (gives 30-day trial): Link: [https://azure.microsoft.com/](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account)

  <img width="1833" height="822" alt="Azure subscription" src="https://github.com/user-attachments/assets/7cad02ce-cce2-4a9f-bec2-355b46f54c21" />

***Note: You will have to sign up and provide valid payment information, but you will be charged Rs.2 for to get verfied.***

### Step-2:  After applying for the subscription and creting  the profile, you can log in at: [https://portal.azure.com](https://portal.azure.com)

  <img width="1883" height="935" alt="azure portal" src="https://github.com/user-attachments/assets/ec225621-a0f9-44b1-b85e-3e819e486412" />


## Part 2 — Creating the Honeypot Virtual Machine

### Step-1: Create Resource Group
- Search for the `Resource Group` and go to that and click on `+Create` to create a new resource group. Give any name to the resource group or follow the below steps and you will get something what shown in the screenshot when you will go on to `Resource Group`.
```
Basics

Resource Group
  └─ Subscription: Azure Subscription 1  
  └─ Resource Group Name: Demo-Lab_1
  └─ Region: South India or Central India
Review + Create
  └─ Create (When Prompted)
```

  <img width="1920" height="955" alt="Demo lab" src="https://github.com/user-attachments/assets/9ff393ff-3c0a-478a-a6c8-58aa0c35e24c" />


### Step-2: Create Virtual Network

- After creating the `Resource Group` ww will create a `Virtual Network` inside the resour group. So search for the `Virtual Network` in the search bar of `Azure Portal` and click on to create and follow the steps as listed below. You can give any name to your `Virtual Network` and after creting go to the `Recource Group' and you will find your `Virtual Network` as shown in the screenshot.
```
Basics

Project Details
  └─ Subscription: Azure Subscription 1  
  └─ Resource Group Name: Demo-Lab_1 
Instance Details
  └─ Virtual Network Name: SOC-Demo-Lab-1
  └─ Region: South India or Central India
Review + Create
  └─ Create (When Prompted)
```

  <img width="1920" height="932" alt="VN Network" src="https://github.com/user-attachments/assets/1fcbcbaf-1a88-4374-8feb-0951ff77d97f" />


### Step-3: Deploy Virtual Machine

- Now again after creating the `Virtual Network` serach for the `Virtual Machine` and click on to create and select the first VM and follow the steps as shown below:
  
  <img width="1898" height="960" alt="VM setting" src="https://github.com/user-attachments/assets/c5ae7266-fb32-4c00-8ac6-701186c2c054" />
  
- Steps For Deploying VM :
```
#Basics

Project Details
  └─ Subscription: Azure Subscription 1  
  └─ Resource Group: Demo-Lab-1 
Instance Details 
  └─ Virtual Machine Name: DIGI-NET-1  
  └─ Region: South India or Central India
  └─ Image: Windows 10 Enterprice, Version 22H2 - x64 Gen2  
Size
  └─ Standard_B1s 1 vCPU, 1 GiB RAM  
Administrator Account
  └─ Username: YourUsername
  └─ Password: YourPassword
  └─ Confirm Password: YourPassword  
Licensing  
  └─ ☑ I confirm I have an eligible Windows 10/11 license

#Disks

OS Disk
  └─ OS Disk Type: Standard HDD

#Networking

Network Interface
  └─ Virtual Network: Create New 
  └─ Subnet: Default 
  └─ Public IP: (new)  
  └─ NIC Network Security Group: Basic
  └─ ☑ Delete public IP and NIC when VM is deleted

#Monitoring

Diagnostics
  └─ Boot Diagnostics: Disable

Review + Create
  └─ Create (When Prompted)
```

- After deploying the VM when you will click on the `Resource Group` you will see something like this shown in the screenshot:
  
  <img width="1920" height="913" alt="VM deployed" src="https://github.com/user-attachments/assets/1a1a840f-88c1-49fb-a50d-6c0a35b98333" />


### Step-4: Configure Network Security Group
- First go to the `Network Security Group(NSG)` in your `Virtual Machine` as shown below:
```
YourVirtualMachine
└─ Networking
  └─ Network Settings
```

- Then we have to `modify the inbound rule` inside of `NSG` but fist delet the `RDP` rule as hown in the screenshot

  <img width="1920" height="909" alt="rdp rule" src="https://github.com/user-attachments/assets/e4baa1f1-0982-403b-a922-42004b59cc9f" />

- Now will add a new inbound rule in `NSG` by following the below steps

  ```
  Add Inbound Security Rule
  └─ Source: Any
  └─ Source Port Ranges: *
  └─ Destination: *
  └─ Service: Custom
  └─ Destination Port Ranges: *
  └─ Protocol: Any
  └─ Action: Allow
  └─ Priority: 100
  └─ Name: DANGER_AllowAnyCustomAnyInbound
  ```
- Below screenshots shows the `Added Inbound Rule` in your `NSG`

  <img width="1920" height="920" alt="new rule" src="https://github.com/user-attachments/assets/da26cc67-575f-437b-8049-7b02f6e8d31e" />

  <img width="1920" height="910" alt="added new rule" src="https://github.com/user-attachments/assets/e2f8804e-6fed-4c1a-8c90-051182ecb2e9" />

- Now `Login` to you `Virtual Machine` using `Remote Desktop Connection` from your system as shown in the screenshot

  <img width="651" height="344" alt="rdp connect" src="https://github.com/user-attachments/assets/13f13880-5ee5-47b1-90cc-6f32078e4db8" />

- Now you have to find the `IP Address` of your VM to login through `RDC`. So, go to your `VM` and you will find the Ip in the `Overview` section like shown in the screenshot

  <img width="1920" height="872" alt="VM IP" src="https://github.com/user-attachments/assets/a0683b22-23f9-43d9-bb86-262d7b529d36" />

- Now we will fill the `IP Address` of your `Virtual Machine` into the `Computer` section of `RDC` and click to connect and then it will ask you to fill up the `User name` and `Password` that you have genrted when creating the `VM` in Azure.
- Now after getting into your `VM` through `Remote Desktop Connection` disable your `Windows Defender Firwall` Path: `Start → wf.msc → Properties → Turn off all firewalls`

  <img width="1875" height="962" alt="Firewall off" src="https://github.com/user-attachments/assets/205b1692-cd93-407b-89fb-9313daa6f324" />


## Part 3 — Generating & Inspecting Logs For Validating Honeypot

### Step-1: Ping your Virtual Machine's IP Address:
- Now we will ping the VM IP address from our local meachine to see that if it is working properly or not as shown in the screenshot

  <img width="1483" height="752" alt="ping" src="https://github.com/user-attachments/assets/be79fdef-6aee-434b-a485-ffc0d635499b" />
  
### Step-2: View Event Logs for Failed Login Attempts
- Now Logout your `Virtual Machine` from `RDC` and try to login `5 to 6 times` with a fake loging credentials to capture `Fake Loging Attempts`. Now again login with the correct login credentials and open the `Event Viewer` and specifically log for `Event Id 4625`. Path for the Event Viewer: `Windows Logs → Security → Event ID 4625` and you will see logs as shown in the screenshot.

  <img width="1808" height="900" alt="brut-force attack" src="https://github.com/user-attachments/assets/67b049b9-576b-466b-b1d3-35adb34d2979" />


***Note: Event ID: 4625 is used to detect failed login attempets***


## Part 4 — Setting Up Log Analytics Workspace (LAW)

In this part, we create a central log storage system using a `Log Analytics Workspace (LAW)` and connect `Microsoft Sentinel` to it. This lets us collect, store, and analyze logs from the honeypot VM. After that we will configure log forwarding using `Azure Monitoring Agent(AMA)` so that all `Windows Security logs—including failed login attempts (Event ID 4625)` are automatically sent from the VM to `Log Analytics` and `Sentinel`.

### Step 1: Create a Log Analytics Workspace (LAW)

- **What it is:**
LAW is a central hub in Azure where all logs from VMs, applications, and services are stored.
- **Why we do this:**
Sentinel cannot work without a LAW because it uses this workspace as its `database`.
- To create a Log Analytics Workspace in Azure, start by searching for "Log Analytics Workspace" in the Azure portal and select it from the results. Click on the "Create" button, then choose a name for your workspace that suits your needs. Follow the steps below to configure `LAW`, and once the deployment is complete, you will see the workspace overview similar to the provided screenshot.

```
  Basics

Project Details
  └─ Subscription: Azure Subscription 1  
  └─ Resource Group: Demo-Lab-1 
Instance Details 
  └─ Name: LAW-DEMO-SOC-LAB
  └─ Region: Central India
Review + Create
  └─ Create (When Prompted)
```

  <img width="1920" height="904" alt="LAW" src="https://github.com/user-attachments/assets/29a07dc9-7ef0-4a29-9472-ba25ea9903ae" />

### Step-2: Deploy Microsoft Sentinel 
- Attach Sentinel to the newly created `LAW`. Just search for the `Microsoft Sentinal` click on to `Create` and you will see the `LAW` and just select and click on to `Add`  as showen in the screenshot.

  <img width="1920" height="923" alt="senti" src="https://github.com/user-attachments/assets/9109ce8b-8744-4ba1-bc83-44d82ecb3b28" />


  <img width="1920" height="709" alt="sentinal 1" src="https://github.com/user-attachments/assets/72254a92-c71b-4595-9dfc-746278ddcce4" />
  

### Step-3: Configure Log Collection Using AMA
- `Azure Monitoring Agent(AMA)` and a `Data Collection Rule (DCR)` are used to forward `Windows Security logs` from the VM into the `Log Analytics Workspace`. Without this `connector + DCR pipeline`, `Sentinel` cannot receive or analyze any security events such as `failed logins (4625) or RDP attempts`.
  
- **Steps:**
   - Open Sentinel
   - Go to Content Hub → Install Windows Security Events (via AMA)
   - Create Data Collection Rule (DCR)
     

  <img width="1920" height="920" alt="AMA" src="https://github.com/user-attachments/assets/714a0b8c-3450-49c1-9e6b-31538c1aa7e6" />

 <img width="1920" height="971" alt="collection rule" src="https://github.com/user-attachments/assets/8e75a089-e253-466b-942c-49f075df17e9" />
 

- **Steps of creating `DCR` is given below:**
  
```
# Basic

Rule Details
  └─ Rule Name: VK-DEMO-1
  └─ Subscription: Azure Subscription 1
  └─ Resource Group: Demo-Lab-1 

# Resources

Source
  └─ ☑ Azure Subscription 1
  └─ ☑ RG-willis
  └─ ☑ YourVictualMachine

# Collect
  └─ All Security Events

# Review + Create
  └─ Create (When Prompted)

```

- Now open `portal.azure.com` in a different tab and navigate to `YourVirtualMachine -> Settings -> Extensions + Applications` to Confirm AMA extension is installed as showen in the screenshot.

  <img width="1920" height="921" alt="AMA agent installed" src="https://github.com/user-attachments/assets/ee63fe3b-900c-43a7-a7fa-5ed27d980f74" />

## Part 5 — Query Logs in Log Analytics (KQL)

- Now we will query the logs in `Log Analytics Workspace` useing `Kusto Query Language (KQL)`. Below are the logs for the initial `1-Hour` after setting up the `Honeypot VM`
- Type `SecurityEvent` and `Run` this query this will show the main table where all `Windows Security Logs` are stored in `Log Analytics Workspace` as showen in the screenshot.

  <img width="1886" height="917" alt="securityevents" src="https://github.com/user-attachments/assets/5d2350ba-616a-4ef5-b827-65f0544ccb0b" />

- Now `Run` the below given query. This KQL query filters the `SecurityEvent` table to show only `Event ID 4625`, which represents failed login attempts on the VM. It then displays key details like `time`, `username`, `computer name`, `event description`, and the attacker’s `IP address` for easy analysis as shown in the screenshot.

```
#KQL

SecurityEvent
| where EventID == 4625
| project TimeGenerated, Account, Computer, EventID, Activity, IpAddress

```

 <img width="1920" height="907" alt="query KQL-1" src="https://github.com/user-attachments/assets/b0130611-7411-481c-b883-5da0952f95cd" />


## Part 6 — Log Enrichment in Microsoft Sentinel Using GeoIP Watchlists

- Since `SecurityEvent` logs only provide the attacker’s `IP address`, we upload a `GeoIP` watchlist to `Sentinel` to enrich those logs with country, city, and latitude/longitude details. This allows us to see where the attacks are coming from and build location-based visualizations like an `attack map`.
- **Steps:**
  - Download the `GioIP` file format it and save it on your system : Use the public dataset: [https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/main/misc/geoip-summarized.csv](https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/main/misc/geoip-summarized.csv) 
  - After downloading the file go to `Azure` platform and navigate to `Microsoft Sentinel -> Configuration -> Watchlist` and click on the link and it will take you to the `Defender Portal` then follow the same path `Microsoft Sentinel -> Configuration -> Watchlist` then will get to add the `GeoIP` as shown in the screenshots.
    
  <img width="1920" height="749" alt="WL-1" src="https://github.com/user-attachments/assets/9559e805-11d9-4471-bebb-22409d321cec" />

  <img width="1920" height="909" alt="wl-2" src="https://github.com/user-attachments/assets/70e8fd55-462f-434e-95a1-7fb9183bcbe0" />

  <img width="1920" height="913" alt="wl-3" src="https://github.com/user-attachments/assets/238e896c-a9fe-4abc-84bb-60cab3d18877" />

  - Now simply select the option to add a new Watchlist, then upload your Excel file directly from your system. Follow the on-screen steps to complete the process. Once the Watchlist is created and uploaded, it will be displayed in your workspace as shown in the screenshot.

```
General
  └─ Name: geoip
  └─ Alias: geoip
Source
  └─ Browse for Files: geoip-summarized (Excel Spreadsheet)
  └─ SearchKey: Network
Review + Create
  └─ Create (When Prompted)
```

  <img width="1799" height="499" alt="wl-4" src="https://github.com/user-attachments/assets/6ca190e8-1cc6-41ac-9765-c16fa7610b45" />

- Now we will run the below `KQL` query to see from where the attacker is coming from same you can see in the screenshot

```
let GeoIPDB = _GetWatchlist("geoip");
SecurityEvent
| where EventID == 4625
| evaluate ipv4_lookup(GeoIPDB, IpAddress, network)
| project TimeGenerated, Account, IpAddress, Country, City, Latitude, Longitude
| order by TimeGenerated desc
```

 <img width="1920" height="854" alt="gioip query" src="https://github.com/user-attachments/assets/fbb3eebf-ae26-434a-a4cf-ed8070ab6027" />



## Part 7 - Creating Sentinel Attack Map
Now we will create a custom Microsoft Sentinel Workbook, which acts as a visualization dashboard for our security data. By adding a query visual and inserting the JSON code, the workbook plots attacker locations using enriched GeoIP data. This allows us to see real-time failed login activity on an interactive world map for easy SOC-style analysis

### Step-1: Creating the Workbook in MS Sentinal

- Return to the `MS Defender` portal where you created the Watchlist. To create a `Workbook` in `Microsoft Sentinel`, navigate through the following path: `Microsoft Sentinel -> Threat Management -> Workbooks`, and then click on `+Add` to start a new workbook. Next, click `Edit`, followed by `Advanced Edit` to open the code editor. Clear the existing content in the editor, paste the provided code, and click "Apply." This will generate the Attack Map, as illustrated in the screenshots


```
# JSON Code

{
	"type": 3,
	"content": {
	"version": "KqlItem/1.0",
	"query": "let GeoIPDB_FULL = _GetWatchlist(\"geoip\");\nlet WindowsEvents = SecurityEvent;\nWindowsEvents | where EventID == 4625\n| order by TimeGenerated desc\n| evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network)\n| summarize FailureCount = count() by IpAddress, latitude, longitude, cityname, countryname\n| project FailureCount, AttackerIp = IpAddress, latitude, longitude, city = cityname, country = countryname,\nfriendly_location = strcat(cityname, \" (\", countryname, \")\");",
	"size": 3,
	"timeContext": {
		"durationMs": 2592000000
	},
	"queryType": 0,
	"resourceType": "microsoft.operationalinsights/workspaces",
	"visualization": "map",
	"mapSettings": {
		"locInfo": "LatLong",
		"locInfoColumn": "countryname",
		"latitude": "latitude",
		"longitude": "longitude",
		"sizeSettings": "FailureCount",
		"sizeAggregation": "Sum",
		"opacity": 0.8,
		"labelSettings": "friendly_location",
		"legendMetric": "FailureCount",
		"legendAggregation": "Sum",
		"itemColorSettings": {
		"nodeColorField": "FailureCount",
		"colorAggregation": "Sum",
		"type": "heatmap",
		"heatmapPalette": "greenRed"
		}
	}
	},
	"name": "query - 0"
}
```

<img width="1920" height="887" alt="Workbook1" src="https://github.com/user-attachments/assets/e90701f3-1a2b-40f0-9037-36d0a2efacb3" />


<img width="1920" height="916" alt="workbook2" src="https://github.com/user-attachments/assets/6ee3e65c-8961-4419-bfb5-357afbc4d38d" />


- The screenshot below displays the `Attack Map` named `Windows VM Attack Map`  generated after visualizing data for one hour, illustrating the detected threat activities and their geographic distribution during this period

<img width="1752" height="841" alt="attack map1" src="https://github.com/user-attachments/assets/de0ab32b-9164-4bc0-8061-e616c1ec7f94" />

- This how `Attack Map` was looking after more than 15+ hours

   <img width="1920" height="900" alt="attack map2" src="https://github.com/user-attachments/assets/d265ea04-2328-4175-ae84-65e0938423a5" />


## Key Skills Demonstrated

- Azure Infrastructure Setup
- Network & Firewall Configuration
- Log Analytics Workspace (LAW)
- Microsoft Sentinel (SIEM) configuration
- Data Collection Rules (DCR)
- AMA connector deployment
- Log enrichment using Watchlists
- KQL querying
- SIEM visualization using Workbooks
- Threat intelligence & correlation analysis


## Conclusion

This project replicates a real-world SOC investigation environment by creating a vulnerable honeypot VM, forwarding logs to Microsoft Sentinel, enriching data via GeoIP intelligence, and creating an interactive attack visualization dashboard.

The repo showcases your ability to:

- Deploy Azure resources
- Configure SIEM tools
- Investigate security events
- Analyze attacks using KQL
- Visualize threat patterns
- A perfect addition to your cybersecurity portfolio.

  






  



  





  

