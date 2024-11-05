# Retail Meta API

## User Guide
Version 3.0

## Introduction

The Retail Meta API empowers Business Unit developers to take full control of integrating their own local retail suppliers, eliminating the need for direct support or resources from the Company Innovation Hub. This API serves as a flexible, self-service framework that gives Business Units the autonomy to manage supplier relationships, configure integrations, and quickly respond to market demands.
By adhering to a standardised set of request and response contracts, Business Units can independently implement supplier connections. This removes the bottleneck of relying on Innovation Hub resources and roadmaps, allowing each unit to prioritise and execute integrations on their own timeline, ensuring faster time-to-market for new offerings.
Ultimately the Retail Meta API puts the power in the hands of Business Units to efficiently scale their retail ecosystem, allowing them to remain agile, competitive, and customer-focused.


**Why should Business Units use it?**

-	**Advanced functionalit**: The Retail Meta API supports a range of the most common supplier action use cases, including acquiring, redeeming, refunding, balance checking and more.
-	**No dependency on Company Innovation Hub**: The Retail Meta API removes dependency on the Company Innovation Hub for supplier integrations, allowing Business Units to integrate with local retail suppliers without delays.
- **Full control over development**: Since the Retail Meta API doesn’t impose restrictions on how integrations are built, development teams have complete autonomy in designing and implementing solutions that meet their specific needs. Business Units have full autonomy to implement integrations within any tech stack.
- **Efficient and targeted integration**: By allowing selective connection to only the necessary endpoints, the Retail Meta API enables development teams to focus on what’s essential, streamlining the integration process, and reducing complexity.
- **Quick time to market and value**: As the Retail Meta API puts the integrations with suppliers in the hands of the Business Units, the time to market is greatly reduced, enhancing the value to Clients. 
 
## Transparent Integration
The key concept behind the Retail Meta API is that it allows for Business Units to  transparently integrate into key Company checkout and voucher flows. From the perspective of the end user (the employee) the system will connect to the relevant local supplier without ever interrupting their user journey.

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail1.png">
  
## Key Capabilities 

The Retail Meta API provides a suite of API-based tools to allow a Business Unit to manage suppliers and products, as well as integrate into the checkout workflow and obtain vouchers for users from BU-managed suppliers.

### Supplier Management

Company considers a supplier to be a third party provider of voucher and eGift codes. When the supplier is managed by the Business Unit, Company considers these to be a “local supplier”. The Company discounts platform makes use of these local suppliers, and their respective API connections in the checkout process when obtaining a voucher code for the purchaser. The Retail Meta API allows a Business Unit to onboard and manage suppliers for their retail offers in a self-service manner. Using the API a Business Unit can:

- Obtain a list of local suppliers for the Business Unit
-	Add a new local supplier
-	Obtain records of configured services and products for a single local supplier
-	Check details of an existing local supplier
-	Remove an existing local supplier
-	Update an existing local supplier					
-	Update credentials for an existing local supplier

## Product Configuration

Company considers a product to be a purchasable item associated with an individual retail offer. These products are linked to the respective supplier that can issue them. The Retail Meta API provides a number of API endpoints to manage products. This gives Business Units complete control over what products are available, and which supplier these products should use.

Using the Retail Meta API a Business Unit can:
			
-	Obtain list of products per local supplier
-	Add a new product for a local supplier
-	Obtain a single product from a given local supplier
-	Suspend a single product for a given local supplier
-	Update a product for a specific local supplier

### Service Action Management
The primary aim of the Retail Meta API is to allow Business Units to integrate directly with suppliers and enable the use of these suppliers in the checkout process. To this end, when a user enters checkout with a voucher from a local supplier, the Company platform will call the Business Unit’s supplier endpoint, as configured through the Retail Meta API, to request the issuance of a voucher through that supplier. Along with issuing vouchers, the Retail Meta API supports a number of different actions, known as services. Each service performs a specific action pertaining to a voucher. 
 
<table>
  <tr style="background-color: #1565C0";
            "color: #FFFFFF">
    <th>Service</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Inventory</td>
    <td>Determine available voucher quantities and denominations (if restricted)</td>
  </tr>
  <tr>
    <td>Dispatch</td>
    <td>Request a new voucher code</td>
  </tr>
  <tr>
    <td>Redeem</td>
    <td>Redeem the voucher token for the voucher itself</td>
  </tr>
  <tr>
    <td>Barcode</td>
    <td>Request the barcode (or barcode URL) for a voucher</td>
  </tr>
  <tr>
    <td>Fund</td>
    <td>Add funds to a reloadable voucher</td>
  </tr>
  <tr>
    <td>Balance</td>
    <td>Request the available balance for a voucher</td>
  </tr>
  <tr>
    <td>Refund</td>
    <td>Request a refund for a voucher</td>
  </tr>
</table>
		

Suppliers may support one or more of these services, but need not support all available service types. For example, voucher suppliers that do not issue any reloadable vouchers may not support the Fund service.
For any given supplier, the Business Unit can decide to connect to as many of the Retail Meta API services as required by the specific local integration.

To assist with self-service administration of the supplier’s available services, the Retail Meta API provides API endpoints that allow the Business Unit to:
-	Add a new service for an existing local supplier
-	Remove a service from a local supplier 
-	Update a service for an existing local supplier

## Service Workflows

### Inventory Service
The Inventory service action allows a voucher provider to inform Company, through the Retail Meta API, of the different voucher denominations available. We support both open and fixed denomination vouchers, and can make use of the Inventory service action to ensure requests to issue vouchers meet the requirements of the supplier.

#### Open vs Fixed Denomination Vouchers
If the Inventory service indicates the supplier issues open denomination vouchers, we will allow users to select any value for the voucher (within the suppliers minimum and maximum limit). If the voucher supplier indicates the voucher has fixed denominations, we will limit options to the denominations the supplier indicates are available (e.g. €10, €20, €50).

Example Workflow

 <img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail2.png">

In the above example, the inventory service is called when a user requests to add a voucher for the product to the cart. The Retail Meta service asks the BU’s supplier endpoint for the voucher inventory restrictions, and returns these to the platform. Should the voucher request be valid, the user can then proceed to checkout.

### Dispatch Service

The Dispatch service is the key service definition for creating new vouchers and eGift codes. The purpose of this service is to ask the local supplier, through the Retail Meta API, to allocate a voucher and return it to the platform.
 
Example Workflow

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail3.png">

In the above example workflow, following confirmation of payment we will make a request to the Business Unit’s supplier endpoint to dispatch a voucher via the local supplier and return it to the Retail Meta API. 
 
### Redeem Service

The Redeem service is a critical service for many local suppliers, particularly if that supplier opts to initially communicate a voucher token to Company, rather than the voucher details themselves. In this context, a voucher token is a placeholder for the actual voucher details; this can be the case when a supplier does not immediately have access to the voucher details and relies on background processes to obtain these. Additionally, for many suppliers this marks the point beyond which the voucher can no longer be refunded. The use of this service is not essential, as some local suppliers may opt to return the voucher code directly to Company during dispatch. If this service is used, the type of response can be varied - some suppliers return the voucher code for Company to render to the member, others return a URL that the member should be directed to in order to view their voucher.

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail4.png">
 
In the above workflow example, the member has attempted to view the voucher code. If Company does not already hold the required details to display the voucher, such as in cases where this information is not communicated during the dispatch service call, we make a request to the Business Unit’s supplier endpoint for this data. This voucher information is returned to Company in the response, and this information is used to show the member the correct voucher code or redirect the member to the page where the voucher details are hosted. The voucher is also marked as “seen”, preventing any subsequent requests to refund this voucher through the interface.

### Barcode Service

The Barcode service is a service used by some local suppliers. The purpose of this service is to obtain the barcode data for the voucher, so it can be displayed on the Company platform. This service allows Company to obtain the correct voucher barcode, or link to the location where the barcode can be viewed.

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail5.png">
 
### Fund Service

For reloadable voucher codes, such as the Instant Reloadable Vouchers offered by Company for various retailers, there are additional services that may be used to manage the balance of a voucher. One such service is the Fund service, which allows Company to add balance to the voucher. This is used in response to a manual checkout for topping up a voucher, or in the case of an automated, recurring purchase (Auto Top-Up) that can be set up in the Company platform.

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail6.png">

In the above example, as the result of a purchase to top up their reloadable voucher, the system makes a call to the Business Unit’s supplier endpoint, which requests that the supplier adds additional balance to the reloadable voucher. The member is then informed whether the balance was successfully added to their voucher.

### Balance Service

The balance service allows the local supplier to communicate the current balance of the voucher code to Company. This is especially useful for reloadable cards, but also can help the member understand what balance is available on their voucher without having to check in-store with the retailer.
 
<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail7.png">
 
In the above example, for a reloadable voucher, the member clicks the “check balance” option within My Account. This triggers a Balance service request to the Business Unit’s supplier endpoint, which in turn requests balance details from the local supplier. Some suppliers opt to directly send the balance back to Company, whereas others instead opt to send a URL to a page where the member can view the balance.

### Refund Service

The refund service facilitates refunds for unused vouchers. This facility is limited to unviewed vouchers and may not be supported by all suppliers. 
 
<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail8.png">
 
In the above example, a refund has been requested by a member for a voucher. The platform requests the refund from the Business Unit’s supplier endpoint, which attempts to request the refund from the local supplier and returns a response via the Retail Meta API to Company. If the refund is successful, the member is informed and any funds will be refunded to their payment card.

## Building a Retail Meta Endpoint

The Retail Meta Endpoint is implemented by the Business Unit and receives incoming requests from Company. The technical implementation of the endpoint is at the full discretion of the Business Unit. Company requires that this endpoint conforms to the request and response contracts shared as part of the Retail Meta API implementation, but allows for full flexibility in technology, hosting, and URL path for these endpoints. Furthermore, individual services may have their own endpoints, or all services may share a single endpoint.

### Endpoint Authentication

The Retail Meta Endpoint operated by the Business Unit should validate that any incoming requests are legitimate via a HMAC signature sent with each request. Should the HMAC signature validation fail, the request should be rejected.

<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>Request example</b>   
   </br>
      <pre>
request = {
    uri: '/retailmeta/example/dispatch',
    payload: {
        'key1': 'value1',
        'key2': 'value2'
    },
    headers: {
        'header1': 'value1',
        'header2': 'value2'
    }
}
secret = ‘123abc456def’
</pre>
</span>
</div>
<div style="flex: 1; padding: 10px;">
<span>
  <b>PHP example</b>
</br>
  <pre>
$uri = str_replace(
    ['%22', '%20'],
    ['', '+'],
    urlencode(mb_strtolower($request->uri))
);
$body = json_encode($request->payload);
$headers = json_encode($request->headers);
$raw = sprintf('%s#%s#%s#', $uri, $payload, $headers);
$signature = hash_hmac('sha256', $raw, $secret);
  </pre>
</span>
</div>
</div>

## Working with the API

### Summary

This section aims to outline requirements and API integration endpoints required for onboarding a supplier and defining their products / services onto the Company Platform.
Environments

<table>
  <tr>
    <th>Environment</th>
    <th>URL</th>
  </tr>
  <tr>
    <td>Production</td>
    <td>https://api.company.net</td>
  </tr>
</table>
 
### Prerequisites

The API integrations mentioned in this document require “Partner” level access to Company. This MUST be requested from Company prior to any integration that may occur.

Following successful partner onboarding, Company will provide three key metadata required:

**Partner Access Token
Partner Refresh Token
Unique Partner ID**

You can find out more about Partner tokens and how to get these here.
<div style="display: flex;">
  <div style="flex: 1; padding: 10px;">
    <span style="color: #1059CD">
      <b>Example Request</b>   
   </br>
      <pre>
http --form POST "https://identity.company.net/access_token" \
    "grant_type=partner" \
    "client_id=$client_id" \
    "client_secret=$client_secret" \
    "partner_id=$partner_id"
      </pre>
    </span>
  </div>
<div style="flex: 1; padding: 10px;">
<span>
  <b>Example response</b>
</br>
  <pre>  
{
    "access_token": "access_token",
    "refresh_token": "refresh_token",
    "token_type": "Bearer",
    "expires_in": 2592000
}
  </pre>
</span>
</div>
</div>

### Onboarding and Managing a Supplier in Company

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail9.png">
 

Onboarding a Supplier into Company can be achieved by using the Company REST API:

<table style="background-color:#808080">
  <tr>
    <th>Create new supplier </br>
POST/retailmeta/suppliers</th>
</tr>
</table>

**Note**: Even though a Supplier has been added, the Supplier will be active for use only after at least 1 product has been added to the Supplier.

Before adding the supplier, you should validate whether the supplier is currently available. Use the following endpoint to do so:
<table style="background-color:#808080">
  <tr>
    <th>Display List of suppliers </br>
GET/retailmeta/suppliers
</th>
</tr>
</table>

To display the records of a supplier by using a Supplier ID:
<table style="background-color:#808080">
  <tr>
    <th>Display records of a single supplier</br>
GET/reatailmeta/suppliers/{supplierId}
</th>
</tr>
</table>

Updating the details for a supplier using a Supplier ID:
<table style="background-color:#808080">
  <tr>
    <th>Update a supplier</br>
PATCH//retailmeta/suppliers/{supplierId}
</th>
</tr>
</table>

Adding the credentials for a supplier that will be stored in the AWS Secrets Manager. 
<table style="background-color:#808080">
  <tr>
    <th>Adds credentials for a supplier</br>
PUT//retailmeta/suppliers/{supplierId}/credentials
</th>
</tr>
</table>

To delete a Supplier, use:
<table style="background-color:#808080">
  <tr>
    <th>Remove a supplier </br>
DELETE/retailmeta/suppliers/{supplierId}
</th>
</tr>
</table> 

### Managing Services for Suppliers in Company
Adding a Service to an existing supplier can be achieved by using the Company REST API:
<table style="background-color:#808080">
  <tr>
    <th>Creates new service</br>
POST/retailmeta/suppliers/{supplierId}/services
</th>
</tr>
</table> 

This endpoint returns a specific service for a particular supplier:
<table style="background-color:#808080">
  <tr>
    <th>Returns a specific service</br>
GET//retailmeta/suppliers/{supplierId}/services/{serviceId}
</th>
</tr>
</table>  

And for deleting a specific service for a selected supplier:
<table style="background-color:#808080">
  <tr>
    <th>Removes a service</br>
DELETE//retailmeta/suppliers/{supplierId}/services/{serviceId}
</th>
</tr>
</table>  
  
### Managing Products for Suppliers in Company

 <img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail10.png">

Adding a Product to an existing supplier can be achieved by using the Company REST API:
<table style="background-color:#808080">
  <tr>
    <th>Creates new product</br>
POST/retailmeta/suppliers/{supplierId}/products
</th>
</tr>
</table>

If you want to display all of the products for a given Supplier, use:
<table style="background-color:#808080">
  <tr>
    <th>Return product list</br>
GET/retailmeta/suppliers/{supplierId}/products
</th>
</tr>
</table>

For finding the details for a single Product for a Supplier:
<table style="background-color:#808080">
  <tr>
    <th>Return a product </br>
GET/retailmeta/suppliers/{supplierId}/products/{productId}
</th>
</tr>
</table> 

If you want to update a Product for a Supplier:
<table style="background-color:#808080">
  <tr>
    <th>Update a product entity </br>
PATCH/retailmeta/suppliers/{supplierId}/products/{productId}
</th>
</tr>
</table> 

For removing a Product from a Supplier:
<table style="background-color:#808080">
  <tr>
    <th>Remove a  product </br>
DELETE/retailmeta/suppliers/{supplierId}/products/{productId}
</th>
</tr>
</table> 

### Storing Supplier Credentials
 
<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail1.png">

When the supplier credentials are added via the API, these are transferred to the AWS Secrets Manager via an integration. 

These credentials can also be managed locally by the Suppliers using their secrets management applications / IAS, but an additional integration will need to be built and managed by the Business Units to act as a middle level integration to extract the key and secrets with the values and added to the APIs as needed.

### Run Mode Guides

#### Rate Limiting
Company APIs rely on rate limits to help provide a predictably pleasant experience for users. Rate limits are applied to all API endpoints regardless of which API you use.

Our current rate limits are set to: 180 requests for a 15 minute window. These limits are per endpoint, per unique ip address. These are also represented through headers in each response returned from the APIs.

<table>
  <tr>
    <th>Header</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>X-Rate-Limit-Limit</td>
    <td>Max allowed requests</td>
  </tr>
  <tr>
    <td>X-Rate-Limit-Remaining</td>
    <td>Remaining requests in the current window</td>
  </tr>
  <tr>
    <td>X-Rate-Limit-Reset</td>
    <td>Time until current window expires</td>
  </tr>
</table>
	
#### Timeouts

The current request / response latency commitment that must be met is 10-15 seconds per timeout.

#### Unhealthy end points

Company employs a circuit breaker to ensure end points are identified and are prevented from being used when too many errors occur. E.g: placing products out of stock when too many errors occur or the supplier is not selectable in case of a large number of errors.

Currently there are two levels at which the circuit breaker will deploy:
-	**Product**: If there are more than 30 errors in 10 minutes period
-	**Supplier**: If there are more than 600 errors in 30 minutes period

#### Idempotency

Unique ID (UID), are present for every item within a transaction with the supplier. These UIDs can be used to debounce multiple attempts and provide protection against accidental duplicate calls causing unintended consequences.

#### Retry Mechanics

Currently there are no retries allowed but work is on to improve upon this along with improvements to the overall user experience in this area. 

For Refunds and reissues of vouchers the Business Units will need to configure the API endpoints at their end. 

#### Conflict Resolution

When a Supplier or Product is deleted, they will no longer be able to be selectable for transactions. The data of the past transactions are stored per Company’s Data Retention Policy. 

#### Response Codes

The following are the Response codes that you would find across the Retail Meta API.

<table>
  <tr>
    <th>Response code</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>200</td>
    <td>Successful operation</td>
  </tr>
  <tr>
    <td>400</td>
    <td>Bad request</td>
  </tr>
  <tr>
    <td>401</td>
    <td>Unauthorised</td>
  </tr>
  <tr>
    <td>404</td>
    <td>Resource not found</td>
  </tr>
</table>

## Other Use Cases

Below is a list of scenarios to illustrate the extended usage of Retail Meta API.

### Supplier Onboarding and Management

Business Units can add multiple suppliers via the Retail Meta API which will be available through a dedicated user interface (UI) in Reward Manager. BUs will be able to use this UI to:
-	Configure a new local supplier integration
-	Manage local supplier credentials stored in AWS Secrets Manager
-	Connect to the different available services
-	Decide which of the available endpoints they would like to use for each local supplier
-	Add additional retailer from the existing local supplier integration

Examples of the Retail Meta API user interface available in Reward Manager:

-	Add new supplier

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail-othuse1.png">

-	Add new supplier service

<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail-othuse2.png">

-	Manage existing supplier integrations
 
<img src="https://github.com/AjayPi/ajaypi.github.io/raw/main/docs/images/metaretail-othuse3.png">

## Further Assistance
If you encounter any issues or need to escalate concerns regarding the Meta Retail API, you can submit a ticket through Zendesk, just as you would for other Company products.

## Glossary of Terms

**Supplier**
Local third-party vendors which the Business Units can engage with to supply goods or services. 

**Retailer**
The vendors where a product (voucher) purchased through RG can be used.

**Offer**
The marketing proposal of a specific retailer to which a certain individual product is associated. E.g.: Purchase this voucher to get 10% off.

**Product**
Purchasable items that can be offered by a retailer.

**Schema**
Schema is a piece of data that is associated with the voucher; examples of schemas are: voucher code, voucher URL.

**Service**
Service in the context of the Retail Meta API is an action that can be requested from the supplier, example: dispatch a voucher, fund a voucher, get the balance of a voucher.

**Circuit Breaker**
The Circuit Breaker design pattern is used to prevent further requests to a third-party system if a service is not working.

**Idempotency**
Idempotency is a property of operations that ensures the same result is produced when an operation is repeated multiple times.

**Response Error**
Three - digit HTTP codes returned by the server in response to a client request. These error codes are identified with 400 responses, for example 400 Bad Request could indicate syntax errors in the requests. 
