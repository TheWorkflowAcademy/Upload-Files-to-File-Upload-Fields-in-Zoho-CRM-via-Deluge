# Upload-Files-to-File-Upload-Fields-in-Zoho-CRM-via-Deluge
A Deluge script for uploading files to a file upload field in Zoho CRM.

## Core Idea
To upload files to a file upload field in Zoho CRM via Deluge, you need to download the files, set the "paramname" parameter as "file" to them, upload them to ZFS via invokeURL, organize the data and update the CRM record. Here's the tutorial and sample code.

## Configuration
Zoho OAuth scopes needed:
* ZohoCRM.files.CREATE
* ZohoCRM.modules.ALL

## Tutorial

### 1. Download the File
First, you need to download the file in your Deluge script. Your file can be from Zoho CRM attachments, WorkDrive, or any other places.
Here's a sample script if you're getting the attachment from a CRM Deal record (assuming there's only 1 attachment there).

```javascript
att = zoho.crm.getRelatedRecords("Attachments", "Deals", dealid);
file = invokeurl
[
	url: "https://www.zohoapis.com/crm/v2/Deals/" + dealid + "/Attachments/" + att.get(0).get("id")
	type: GET
	connection:"crm_oauth_connection"
];
info file;
```

### 2. Set the Paramname as "file" to the File
Then, apply "setParamname("file") on the file variable.
* The "setParamName" built-in function is used to set the specified name for the file object that needs to be sent in multipart form-data using invokeUrl later.
```javascript
file.setparamname("file_name");  
```
_Replace "file_name" with a relevant name or use the getFileName() function to get the file name of your file_

### 3. Upload the file to ZFS (Zoho File Systems) via invokeURL
Before updating the file to the file uplaod field on CRM, we first need to upload it to ZFS. Then, get the file ID.
* [Read more about uploading files to ZFS here.](https://www.zoho.com/crm/developer/docs/api/upload-files-to-zfs.html)
```javascript
resp = invokeurl 
[ 
	url: " https://www.zohoapis.com/crm/v2/files" 
	type: POST 
	files: file 
	connection: "crm_oauth_connection" 
]; 
fileid = resp.get("data").get(0).get("details").get("id"); 
```

### 4. Organize the Data and Update the Field
You now have everything you need for the update. Just organize the data, and update the field!
* Put the file ID in a map with "file_id" as the key
* Put that map in a list
* Put the the list in your update map with the field name as the key
* Run the update with the usual Deluge update task.
```
fmp = Map(); 
fmp.put("file_id",fileid); 
fileList = List();
fileList.add(fmp); 
mp = Map(); 
mp.put("Final_Design",fileList); 
update = zoho.crm.updateRecord("Deals", dealid, mp);
info update;
```
*Note: You can attach more than one file to a file upload field. Just add to the "fileList" variable accordingly.*
