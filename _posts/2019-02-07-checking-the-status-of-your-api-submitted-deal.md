---
layout: post
title:  "Checking the Status of Your API Submitted Deal"
date:   2019-02-07 4:00:00 -0400
categories: api
author: Kyle Chan
---

*Now that you have submitted your first deal via our [Lead Submission Endpoint](http://tech.forwardfinancing.com/api/2017/03/07/using-our-lead-submissions-api.html),
you're probably wondering how it's doing.*

## Getting Started
The only things you'll need are 
- The **lead ID** you received when you first submitted your deal
- Your **API key**

#### Authentication

You will need to contact us to get your **API key**. The API key should be included
in your request headers such that the request header key is `api_key` and the
request header value for that key is your **API key**. If you omit the api_key or
provide an incorrect one, the HTTP status code of the response will be 401
Unauthorized.

#### HTTPS

Since your API requests will contain sensitive personal information about our
customers, you must use SSL (HTTPS). To discourage insecure API requests,
the response code will be 403 Forbidden for all requests made over HTTP.

#### Rate Limiting

We impose a rate limit per user of 1 request per second. If you exceed this rate
of requests, the response code will be 429 Too Many Requests. If you need to
exceed this limit for some reason, please let us know!

## The Deal Status Endpoint

You will be able to check your deal status with a GET request to the `/v1/deal_status` 
endpoint with your **lead ID**.
For example, the GET url might be

`https://api-staging.forwardfinancing.com/v1/deal_status/<LEAD_ID>`

### Request

The only thing you'll need in the request header is your **API key**!

```
api_key: YOUR_API_KEY_HERE
```

### Response
The possible response body fields are 
- stage
- maxApproval
- maxPayments
- offerLink
- declineDrivers
- declineNotes
- missingInfo

Your response will contain different fields depending on the status of your deal.
All responses will contain the **stage** of your deal.

|**Deal Status**|stage  | maxApproval   | maxPayments  | offerLink  | declineDrivers  | declineNotes  | missingInfo |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   |   |   |   |   |   |   |   |
| We're currently processing your deal  | x |  |  |   |   |   |   |
|   |   |   |   |   |   |   |   |
| Your deal has been approved  | x | x | x | x |   |   |   |
|   |   |   |   |   |   |   |   |
| Your deal has been declined  | x |   |   |   | x | x |   |
|   |   |   |   |   |   |   |   |
| We need more information  | x |   |   |   |   |   | x |

## Glossary
**stage** - the current state of your submitted deal in our process 

**maxApproval** - the maximum advance amount approved for this merchant

**maxPayments** - the maximum number of payments approved for this merchant (term in days), (we arrived that by multiplying months by 20) 

**offerLink** - link to our offer calculator where you can select terms and request contracts

**declineDrivers** - reasons why this deal was declined chosen from a set list (see below for list)

**declineNotes** - additional explanation for why this deal was declined 

**missingInfo** - additional information that is needed for a decision to  be made on this deal


## Sample Responses 
These are some possible responses you may receive from our API
```
{
  "stage": "Approval Sent",
  "maxApproval": "10000",
  "maxPayments": "10",
  "offerLink": "offer.com"
}
```

```
{
  "stage": "Declined",
  "declineDrivers": [Default history, Excessive liens or judgements],
  "declineNotes": "This merchant does not look credible"
}
```

```
{
  "stage": "File Missing Info",
  "missingInfo": "We're going to need more recent bank statements please!"
}
```

```
{
  "stage": "Working"
}
```

# Possible Decline Drivers

  - Application Incomplete
  - Challenged personal credit history
  - Default history
  - Excessive liens or judgements
  - Excessive negative days or NSFs
  - Excessive recent inquiries
  - Existing customer
  - Failed merchant interview
  - Fraud
  - Inability to extend a minimum offer
  - Low adjusted deposit count
  - Low adjusted deposit volume
  - Open or recent bankruptcy
  - Over daily submission limit
  - Over-leveraged w/ pre-existing financing
  - Personal Bank Statements
  - Poor business risk score
  - Poor month to date performance
  - Poor personal risk score
  - Presence of criminal filings
  - Previous decline
  - Prohibited business type
  - Restricted business location
  - Poor Industry Risk Score
  - Short-time In business
  - Sub 500 FICO
  - Too early to net 50%
  - Unexplained UCCs
  - Unwillingness to pay creditors
  - Volatile deposits

