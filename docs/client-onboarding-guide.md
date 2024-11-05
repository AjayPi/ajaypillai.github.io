
## Client Onboarding Integration

Integrating with the "Company" Platform for onboarding clients and members.

## Summary
This document aims to outline requirements and API integration endpoints required for onboarding a client and respective members onto the Company Platform.

## Environments

<table style="border-radius: 10px";
              "border: 5px solid #154360";>
<tr>
<th>Environment</th>
<th>URL</th>
<tr>
<td>Testing:</td>
<td>https://api.testing.aws.company.net</td>
</tr>
<td>Production:	</td>
<td>https://api.company.net</td>
</tr>
</table>

## Prerequisites
The API integrations mentioned in this document require “Partner” level access to Company. This MUST be requested from Company prior to any integration that may occur.

Following successful partner onboarding, Company will provide three key metadata required:


<ul>
<li>Partner Access Token</li>
<li>Partner Refresh Token</li>
<li>Unique Partner ID</li>
</ul>

## Onboarding a Client into Company

Onboarding a client into Company can be achieved by using the Company REST API.

## Create a programme

<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>SHELL</b>   
   </br>
	    
      <pre>
      curl --location 'https://api.company.net/scheme' \
    --header 'Accept: application/json' \
    --header 'Content-Type: application/json' \
    --header 'Authorization: Bearer {$partner_token} \
    --header 'Accept: application/vnd.company+json;version=3.0' \
    --data '{
   "name": "Example Programme Name",
   "companyName": "Test Company A",
   "companyLegalName": "Test Company A Ltd.",
   "locale": "BE",
	 "accountManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
   "implementationManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
   "externalAccountId": "0019E00002NyByYQAY",
   "externalParentAccountId": "0021E05009MoByYRT1",
	 "externalLinkFormat": "https://sf.com/{scheme.externalId}",
   "template": {
        "schemeId": "d6b933ea-b5f5-48cb-9fce-d105902ae5ed",
        "features": {
            "layoutName": "Reward Hub Layout",
            "additionalLanguages": "en_GB|fr_FR|de_DE",
            "loginConfiguration": true
        }
    }
}
</pre>
</span>
  </div>
  
  <div style="flex: 1; padding: 10px;">
<span>
  <b>Response</b>
</br>
  <pre>
  {
   "uuid": "385289a3-b375-4b9e-89f8-ce6d08c449bd",
   "name": "Example Programme Name",
   "companyName": "Test Company A",
   "companyLegalName": "Test Company A Ltd.",
   "schemeType": "integrated",
   "localeId": 46,
   "locale": "BE",
	 "accountManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
   "implementationManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
   "status": -1,
   "type": 0,
   "billingCountry": 46,
   "externalLinkFormat": "https://sf.com/{scheme.externalId}",
   "externalAccountId": "0019E00002NyByYQAY"
}
</pre>
</span>
  </div>
  </div>  


When a request for programme creation is sent, the response contains a programme status:

<ul>
<li>-1 - programme is not provisioned / not ready for use</li>
<li>0 - Implementation - programme is prepared for use and configuration</li>
<li>1 - Live - programme’s Reward Hub is accessible for members</li>
<li>2 - Closing - the programme is prepared to be closed after 60 days</li>
<li>3 - Closed - programme is not available</li>

Before using the programme, you should validate its status is different from -1. Use the following endpoint:

### Retrieve programme details
<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>SHELL</b>   
   </br>
	    
      <pre>
      curl --location 'https://api.company.net/schemes/{scheme_uuid}' \
    --header 'Accept: application/json' \
    --header 'Content-Type: application/json' \
    --header 'Authorization: Bearer {$partner_token} \
    --header 'Accept: application/vnd.company+json;version=3.0'
    </pre>
    </span>
    </div>

    <div style="flex: 1; padding: 10px;">
<span>
  <b>Response</b>
</br>
  <pre>
  {
    "uuid": "385289a3-b375-4b9e-89f8-ce6d08c449bd",
    "name": "Example Programme Name",
    "companyName": "Test Company A",
    "companyLegalName": "Test Company A Ltd.",
    "addressLine1": "Address Line 1",
    "addressLine3": "Address Line 3",
    "postalCode": "SW18 1UX",
    "billingCountry": "Company BE",
    "accountManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
    "implementationManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
    "status": 0,
    "branding": {
        "colours": {
            "navigationBackground": "003865",
            "navigationText": "ffffff",
            "buttonBackground": "003865",
            "buttonText": "ffffff"
        },
        "favicon": "https://static.company.net/img/COLLATERAL_PLACEHOLDER.jpg"
    }
}
  </pre>
</span>
</div>
</div>

### Add or replace the current domain attached to a programme (programme alise)
<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>SHELL</b>   
   </br>
      <pre>
      curl --location
--request PUT 'https://api.company.net/scheme/{scheme_uuid}/alias' \
--header 'Accept: application/vnd.company+json;version=3.0' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {$partner_token} \
--data '{
	"hostname": "site408871.company.net"
}'
</pre>
</span>
</div>
<div style="flex: 1; padding: 10px;">
<span>
  <b>Response</b>
</br>
  <pre>
	  {
    "hostname": "site408871.company.net"
}
  </pre>
</span>
</div>
</div>

### Update a programme's domain
<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>SHELL</b>   
   </br>
      <pre>
      curl --location
--request PUT 'https://api.company.net/scheme/{scheme_uuid}/alias' \
--header 'Accept: application/vnd.company+json;version=3.0' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {$partner_token} \
--data '{
	"hostname": "site408871.company.net"
}'
      </pre>
    </span>
  </div>
  <div style="flex: 1; padding: 10px;">
<span>
  <b>Response</b>
</br>
  <pre>
  {
    "hostname": "site408871.company.net"
}
  </pre>
</span>
  </div>
</div>

### Edit programme details

<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>SHELL</b>   
   </br>
      <pre>
      curl --location --request PUT 'https://api.company.net/schemes/{scheme_uuid}' \
--header 'Accept: application/vnd.company+json;version=3.0' \
--header 'Content-Type: application/json;charset=utf-8' \
--header 'Authorization: Bearer {$partner_token}' \
--data '{
	"companyLegalName": "Test Company A Ltd.",
	"postalCode": "NW10 7HQ",
	"addressLine1": "Address Line 1",
	"addressLine2": "Address Line 2",
	"addressLine3": "Address Line 3",
  "status": 1
  "accountManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
  "implementationManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef"
}'
      </pre>
    </span>
  </div>
<div style="flex: 1; padding: 10px;">
<span>
  <b>Response</b>
</br>
  <pre>
  {
	"companyLegalName": "Test Company A Ltd.",
	"postalCode": "NW10 7HQ",
	"addressLine1": "Address Line 1",
	"addressLine2": "Address Line 2",
	"addressLine3": "Address Line 3",
  "status": 1,
  "accountManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef",
  "implementationManagerId": "afebf833-24e1-4feb-9545-5a587210f3ef"
}
  </pre>
</span>
</div>
</div>



> [!NOTE]  
> Programme status can be changed only from implementation (0) to live (1), from live (1) to closing (2) and from closing (2) to closed (3). Other combinations are not allowed.

## Programme Branding

### Update a programme's logo
<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>SHELL</b>   
   </br>
      <pre>
      curl --location 'https://api.company.net/scheme/{scheme_uuid}/logo' \
--header 'Accept: application/vnd.company+json;version=3' \
--header 'Content-Type: multipart/form-data' \
--header 'Authorization: Bearer {$partner_token}' \
--form 'image=@"/some/path/1050x300-487479.png"'
      </pre>
    </span>
  </div>
  <div style="flex: 1; padding: 10px;">
<span>
  <b>Response</b>
</br>
  <pre>
  {
    "imageUrl": "https://static.company.net/img/1050x300-487479.png",
    "imageSVG": "https://static.company.net/img/COLLATERAL_PLACEHOLDER.svg",
    "companyImage": "https://static.company.net/img/COLLATERAL_PLACEHOLDER.jpg",
    "companyImageSVG": "https://static.company.net/img/COLLATERAL_PLACEHOLDER.svg"
}
  </pre>
</span>
</div>
</div>

### Update a programme's branding color
<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>SHELL</b>   
   </br>
      <pre>
	      curl --location --request PUT 'https://api.company.net/scheme/{scheme_uuid}/colour' \
--header 'Accept: application/vnd.company+json;version=3' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {$partner_token}' \
--data '{
	"navigationBackground": "1d5f94",
	"buttonBackground": "1d5f94"
}'
      </pre>
    </span>
  </div>
<div style="flex: 1; padding: 10px;">
<span>
  <b>Response</b>
</br>
  <pre>
  {
	"navigationBackground": "1d5f94",
	"buttonBackground": "1d5f94"
}
  </pre>
</span>
</div>
</div>

 
### Onboarding a Member into a Company Client
Onboarding members into Company can be achieved by using the Company SCIM API.
