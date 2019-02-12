---
layout: post
title:  "Partner API Status Endpoint"
date:   2019-02-07 4:00:00 -0400
categories: api
author: Kyle Chan
---

*Now that you have submitted your first deal via our [Lead Submission Endpoint](http://tech.forwardfinancing.com/api/2017/03/07/using-our-lead-submissions-api.html),
you're probably wondering how it's doing.*

## Getting Started
The only things you'll need are: 
- The lead `id` you received when you first submitted your deal
- Your `api_key`

### Authentication

You will need to contact us to get your `api_key`. The API key should be included
in your request headers such that the request header key is `api_key` and the
request header value for that key is your `api_key`. If you omit the api_key or
provide an incorrect one, the HTTP status code of the response will be 401
Unauthorized.

### HTTPS

Since your API requests will contain sensitive personal information about our
customers, you must use SSL (HTTPS). To discourage insecure API requests,
the response code will be 403 Forbidden for all requests made over HTTP.

### Rate Limiting

We impose a rate limit per user of 1 request per second. If you exceed this rate
of requests, the response code will be 429 Too Many Requests. If you need to
exceed this limit for some reason, please let us know!

## The Deal Status Endpoint

You will be able to check your deal status with a GET request to the `/v1/deal_status/LEAD_ID` 
endpoint with the `id` of your lead.
For example, the GET url might be

`https://api-staging.forwardfinancing.com/v1/deal_status/<LEAD_ID>`

### Request

The only thing you'll need in the request header is your **API key**!

```
  api_key: "YOUR_API_KEY_HERE"  
```

#### Sample Request
With CURL: 
```
curl -X GET \
  'https://api-staging.forwardfinancing.com/v1/deal_status/LEAD_ID_GOES_HERE' \
  -H 'api_key: HERE_IS_MY_API_KEY'
```

### Response
The possible response body fields are 
- `stage` - the current state of your submitted deal in our process 
- `max_approval` - the maximum advance amount approved for this merchant
- `max_payments` - the maximum number of payments approved for this merchant (term in days)
- `offer_link` - link to our offer calculator where you can select terms and request contracts
- `decline_drivers` - reasons why this deal was declined chosen from a set list (see below for list)
- `decline_notes` - additional explanation for why this deal was declined 
- `missing_info` - additional information that is needed for a decision to  be made on this deal

Your response will contain different fields depending on the status of your deal.
All responses will contain the **stage** of your deal.

|**Deal Status**|stage  | maxApproval   | maxPayments  | offerLink  | declineDrivers  | declineNotes  | missingInfo |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Processing | ✔ |  |  |   |   |   |   |
| Approved  | ✔ | ✔ | ✔ | ✔ |   |   |   |
| Declined  | ✔ |   |   |   | ✔ | ✔ |   |
| Missing Information  | ✔ |   |   |   |   |   | ✔ |

## Response Codes
**200 Success** - Your deal was found successfully and it's details are in the response body.
 
**401 Unauthorized** - Please provide a valid `api_key` header. You can contact us to obtain an `api_key`.

**403 Forbidden** - Please make your request over HTTPS. We do not support requests over insecure HTTP.

**429 Too Many Requests** - We limit the amount of requests to  1 request per second. If this is insufficient for your use case, let us know!

## Sample Responses 
These are some possible responses you may receive from our API
```
{
  "stage": "Approval Sent",
  "max_approval": 10000,
  "max_payments": 20,
  "offer_link": "offer.com"
}
```

```
{
  "stage": "Declined",
  "decline_drivers": "Default history;Excessive liens or judgements",
  "decline_notes": "This merchant does not look credible"
}
```

```
{
  "stage": "File Missing Info",
  "missing_info": "We're going to need more recent bank statements please!"
}
```

```
{
  "stage": "Working"
}
```

# Example Decline Drivers

  - Application Incomplete
  - Challenged personal credit history
  - Default history
  - Excessive liens or judgements
  - Failed merchant interview
  - Fraud
  - Open or recent bankruptcy
  - Over-leveraged w/ pre-existing financing
  - Poor business risk score
  - Poor month to date performance
  - Poor personal risk score
  - Presence of criminal filings
  - Previous decline
  - Volatile deposits

