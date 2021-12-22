
# OpenScan Cloud API
Photogrammetry Web API

## Overview:
The OpenScan Cloud is intended to be a decentralized, open, and free photogrammetry web API. 

The API can be used as a great addition to existing photogrammetry rigs like the OpenScan Classic or Mini, but also any other rig. To get started you'll need a a token (public or private) and a set of images.

There are currently four API implementations that you can use to generate 3D models through photogrammetry:
- Desktop Uploader for Windows
- Beta Firmware for the OpenScan Classic & Mini
- Python Uploader 
- REST API

**Note:** The application is free (and hopefully I can keep it free / donation-based in the future). Your data will be transferred through Dropbox and stored / processed on personal servers. The sets of images and processed 3D models are used for further research but not published outside without explicit consent!

If you like the project, feel free to support my work on [BuyMeACoffee](https://www.buymeacoffee.com/OpenScan)!

## Implementations

### #1 Desktop Uploader for Windows - [Download](https://github.com/OpenScanEu/OpenScanCloud/raw/main/uploader/Uploader.win.zip)
The uploader is a standalone *.exe* which can be run on any Windows machine. It allows you to select either a folder or a zip file containing images and uploading those to the cloud which initializes the process. 

All you need is an individual token and a set of photogrammetry images. 

#### KNOWN ISSUE: Unfortunately, for some users the software might time-out when establishing a connection to my servers. This can be identified if it does not change for several minutes... Sorry :) I will try my best to release and update as soon as possible.
![Desktop Uploader for Windows](https://i.imgur.com/jUSTf1o.png)

### #2 Beta Firmware for the OpenScan Classic & Mini - [Sourecode](https://github.com/OpenScanEu/OpenScanCloud/blob/main/uploader/firmware_beta.json)
You can install the latest firmware for the OpenScan Classic and Mini by running the following command:

```sudo wget -O /home/pi/.node-red/flows_raspberrypi.json https://raw.githubusercontent.com/OpenScanEu/OpenScanCloud/main/uploader/firmware_beta.json && node-red-restart```

You will need to reboot the Raspberry Pi after the install and wait 1 - 2 minutes.

![Firmware for OpenScan Pi](https://i.redd.it/yrmopdyr9ts71.png)

### #3 Python Uploader - [Sourecode](https://github.com/OpenScanEu/OpenScanCloud/blob/main/uploader/uploader.py)
The python script can be a starting point to create your own solution with Python. 

Currently you only need to change a handful of parameters at the beginning of the file. These are documented in the source code.

**Note:** the _requests_ module is needed (```pip install requests```)

### #4 REST API
Lastly you're also able to use the OpenScanCloud RestAPI which can be a starting point to create a custom solution. 

The REST API mostly uses HTTP GET requests with query string parameters and JSON responses. 

Request authentication is used via HTTP basic authentication using the *Authorization* request header. 

The username for the basic authentication is *openscan* and the password is *free*. Both need to be combined and encoded using Base64 to be a valid *Authorization* value.

The mentioned endpoints are on the _openscanfeedback.dnsuser.de_ (port 1334) domain. 

The upload endpoints have a file size limit of 200 MB and therefor project files that are larger than 200 MB need to be split into multiple parts.

----

- ### /createProject
Initializes a new project. 

**Note:** You cannot use an existing project name.

**Structure:**

    http://openscanfeedback.dnsuser.de:1334/createProject

You need to pass five query string parameters with this endpoint. You can find the parameters down below with an example.

|query string|description|
|--|--|
|token|Your personal token|
|project|The new project name|
|photos|The amount of photos that will be provided|
|parts|How many upload parts will be file be split in|
|filesize|The exact size in bytes|

    /createProject(token, project, photos, parts, filesize)

**Example:**

`http://openscanfeedback.dnsuser.de:1334/createProject?token=yourtoken&project=yourprojectname&photos=youramountofphotos&parts=youramountofuploadparts&filesize=yourfilebytesize`  

**Response:**
The server will respond with a 200 OK success status response code when the request is processed. 

The response will contain a temporary upload link which can be used to send the photogrammetry images.

`status 200 {'status':'created', 'ulink':[youruploadlinks], 'credit':yourcreditsused}`

---

- ### /startProject
Starts the project. 

**Note:** This is required after successfully uploading the files to initialize the processing of the image-set.

**Structure:**

    http://openscanfeedback.dnsuser.de:1334/startProject

You need to pass two query string parameters with this endpoint. You can find the parameters down below with an example.

|query string|description|
|--|--|
|token|Your personal token|
|project|The created project name|

    /startProject(token, project)

**Example:**

`http://openscanfeedback.dnsuser.de:1334/startProject?token=yourtoken&project=yourprojectname`  

**Response:**
The server will respond with a 200 OK success status response code:

`status 200 {'status':'initialized'}`

---

- ### /getProjectInfo
Provides the status of the started project.


**Structure:**
    
    http://openscanfeedback.dnsuser.de:1334/getProjectInfo

You need to pass two query string parameters with this endpoint. You can find the parameters down below with an example.

|query string|description|
|--|--|
|token|Your personal token|
|project|The created project name|

    /getProjectInfo(token, project)

**Example:**

`http://openscanfeedback.dnsuser.de:1334/getProjectInfo?token=yourtoken&project=yourprojectname`  

**Response:**
The server will respond with a 200 OK success status response code. There are three status codes: *initialized*, *processing started*, and *processing done*.

**Note:** This response includes the upload and download links (if available). 

   `status 200 {'dlink':downloadlink, 'status':status, 'ulink':uploadlinks}`

---

- ### /getTokenInfo
Provides information regarding the specified token such as the credit amount and limitations regarding file-sizes and photo counts.

**Structure:**
    
    http://openscanfeedback.dnsuser.de:1334/getTokenInfo

You need to pass the token as query string parameter with this endpoint. You can find the parameter down below with an example.

|query string|description|
|--|--|
|token|Your personal token|

    /getTokenInfo(token, project)

**Example:**

`http://openscanfeedback.dnsuser.de:1334/getTokenInfo?token=yourtoken`  

**Response:**
The server will respond with a 200 OK success status response code. 

   `status: 200 {"credit":24468629501,"limit_filesize":2000000000,"limit_photos":1000}`

### Uploading Images
When you initialized your new project you receive an upload link where you're able to upload your photogrammetry pictures.

It's recommened to upload a ZIP file.

You need to send files as binary data through a POST request with an *application/octet-stream* as *Content-type* header. You don't need an *Authentication* header.

**Note:** Upload links are valid for 4 hours.

**Example:**

`https://content.dropboxapi.com/apitul/1/youruniqueidentifier`

**Response:**

When successfully uploaded you receive a 200 OK status response code. After this you're able to start your project.

    status: 200
    body: {"content-hash": "yourcontenthash"}

### Response Troubleshooting

Below you will find two of the common error status response codes and what they might mean:

The server will respond with a 401 unauthorized status response code when the request lacks valid authentication.

`{"error":"Unauthorized access"}`

The server will respond with a 400 bad request status response code when the request cannot be processed by the server. This is possibly due to a wrong token, existing project name, etc.

    "The browser (or proxy) sent a request that this server could not understand."
    

## Token and Credit System
### Credit
Credit is used to monitor the overall usage of processing resources. The credit value is bound to each token.

### Tokens
- **Public Token**
There will be a public token, with a certain amount of credit per time (e.g. 10 GB per day or so). People using this token won't have access to additional features like auto-email when the reconstruction is done + individual support. Furthermore the data submitted through this token will be used for further research and improvement of the processing engine (and future features of OpenScan). In case of high loads on the server side, sets created with private tokens might be prioritized (at some point in the future)

- **Private Token**
This token is bound to an individual and certain details (first name, surname, email address) in order to allow additional features, like email alerts and individual support. At the current stage, the image sets submitted will be used for internal research and testing. No images/3d models will be published.

### Apply for Private Token
To apply for a private token you should emal to cloud (at) openscan.eu and provide the following information:
- Name
- Email

## Changelog
- 2021-12-20 added Texture to the 3d model + improved firmware
- 2021-10-11 added Beta Firmware for OpenScanPi 
- 2021-10-08 added a Windows Uploader GUI in /uploader

## Donate
If you like the project, feel free to support my work on [BuyMeACoffee](https://www.buymeacoffee.com/OpenScan)!
