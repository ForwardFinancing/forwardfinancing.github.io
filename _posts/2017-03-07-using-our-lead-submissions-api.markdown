---
layout: post
title:  "Using Our Lead Submissions API"
date:   2017-03-07 12:00:00 -0400
categories: api
author: Zach Cotter
---

*Our Lead Submissions API is designed to allow our business partners to submit
potential deals automatically from their system to ours.*

## Getting Started

Check out our automatically generated API docs
[here](https://api-staging.forwardfinancing.com/api_docs). They are
interactive, so you can send example requests directly from the documentation in
addition to viewing the request and response formats. We also have a [
 github repo containing example client code in various languages
](https://github.com/ForwardFinancing/partner_api_client_samples).

#### Authentication

You will need to contact us to get your API key. The API key should be included
in your request headers such that the request header key is `api_key` and the
request header value for that key is your API key. If you omit the api_key or
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

## The Lead Endpoint

Lead Submissions may be made via POST request to the `/v1/lead` endpoint.
For example, the POST url might be
`https://api-staging.forwardfinancing.com/v1/lead`

### Request

The request body should be in JSON format, so the content type header should
set to JSON as well. Your headers should look like:

```
api_key: YOUR_API_KEY_HERE
Content-Type: application/json
```

The request body should contain a JSON payload that describes the lead you are
submitting. The full format can be found in our API documentation. An example
request body is below:

```
{
  "lead": {
    "contacts_attributes": [
      {
        "first_name": "string",
        "last_name": "string",
        "email": "test@forwardfinancing.com",
        "title": "string",
        "born_on": "2015-01-01",
        "home_phone": "6176781000",
        "cell_phone": "6176781000",
        "ssn": "234345566",
        "ownership_date": "2015-01-01",
        "current_address_attributes": {
          "street1": "string",
          "street2": "string",
          "city": "string",
          "state": "AK",
          "zip": "00112"
        }
      }
    ],
    "account_attributes": {
      "entity_type": "Sole Proprietor",
      "name": "string",
      "started_on": "2015-01-01",
      "legal_name": "string",
      "phone": "6176781000",
      "email": "test@forwardfinancing.com",
      "website": "string",
      "fein": "string",
      "monthly_revenue": "Less than $5,000",
      "industry_name": "Laundry and dry cleaning services",
      "current_address_attributes": {
        "street1": "string",
        "street2": "string",
        "city": "string",
        "state": "AK",
        "zip": "00112"
      }
    },
    "loan_attributes": {
      "company_name": "string",
      "daily_payment_amount": 0,
      "balance": 0
    },
    "application_attributes": {
      "has_current_loan": true,
      "applicant_is_owner": true,
      "loan_use": "Debt Refinancing",
      "capital_needed": "string",
      "owner_1_percent_ownership": 0,
      "owner_2_percent_ownership": 0,
      "reference_id": "string"
    }
  }
}
```

### Response

If your request is unsuccessful, for example if the JSON is invalid or certain
required attributes are missing, you will receive HTTP status code 422
Unprocessable Entity. The response body may contain information about what part
of your request was invalid in JSON format, for example:

```
{
  "errors": {
    "contacts_attributes": [
      "Must provide address"
    ]
  }
}
```

If your request is successful, you will receive HTTP status code 201 Created.
The response body will contain a success message, and an ID for your submission,
which will be used later for submitting attachments. For example

```
{
  "message": "Your lead was successfully created.",
  "id": "QTEQ.YJNyDzbMP0YX0D7qQr7PZcn7CU2E.TIIvFdZlr._ps.FDvDvwqGzI-c5FQ"
}
```

## The Attachment Endpoint

The attachment endpoint enables you to submit documents related to leads that
have already been submitted. Attachment Submissions may be made via POST
request to the `/v1/attachment` endpoint. For example, the POST url might be
`https://api-staging.forwardfinancing.com/v1/attachment`. To submit multiple
attachments, submit multiple requests.

You can send attachments by submitting either the raw binary of the attachment
file, or by submitting a URL to the attachment.

### Request - Sending the Attachment as a Binary

#### URL Parameters

The attachments endpoint requires two URL parameters:

1) The `filename` parameter, which indicates the desired filename in our system
for the attachment you are uploading. For example `bank_statement.pdf`.

2) The `lead_id` parameter, indicating which of your previously submitted leads
the attachment is associated with. The lead id can be found in the response to
the leads endpoint. For example
`QTEQ.YJNyDzbMP0YX0D7qQr7PZcn7CU2E.TIIvFdZlr._ps.FDvDvwqGzI-c5FQ`.

The full url with parameters will look something like:
`https://api-staging.forwardfinancing.com/v1/attachment?filename=bank_statement.pdf&lead_id=QTEQ.YJNyDzbMP0YX0D7qQr7PZcn7CU2E.TIIvFdZlr._ps.FDvDvwqGzI-c5FQ`

If any of the required URL parameters are omitted or invalid, you will receive
a 400 Bad Request.

#### Headers

As with the other endpoints, you must include your API Key in the `api_key`
header. Since the request body format for this endpoint is not JSON, you should
NOT set the `Content-Type` to `application/json`

#### Body

The request body should contain the binary of the file you want to upload. In
most languages, you can accomplish this by reading the file contents into memory
and then outputting them into your request body as binary or a string.

If for some reason, your system does not allow you to send a binary request
body, you can Base 64 encode the binary into a string. If you choose this
option, you must indicate the body is encoded by passing `encoded=true` in the
URL parameters. Be sure to use strict Base 64 encoding, which encodes new line (`\n`) characters.

### Request - Sending a URL to the attachment

The same endpoint can be used to submit a URL to the attachment, rather than
the raw binary.

#### URL Parameters

No URL parameters are needed, but optionally you can include the body params in
the URL instead.

#### Headers

As with the other endpoints, you must include your API Key in the `api_key`
header. The request body format is JSON, so the `Content-Type` should be
`application/json`

#### Body

Three JSON body params are required. In addition to the filename and lead_id
parameters, you provide an `attachment_url` parameter, which is a URL to the
attachment you want to submit. For example:

```
{
  "attachment_url": "https://yourwebsite.com/files/some_attachment.pdf",
  "filename": "bank_statement.pdf",
  "lead_id": "QTEQ.YJNyDzbMP0YX0D7qQr7PZcn7CU2E.TIIvFdZlr._ps.FDvDvwqGzI-c5FQ"
}
```

### Response

If the request is successful, you will receive a response with an HTTP status
code of 202 Accepted, which indicates that the attachment is being processed
into our system.

```
{
  "message": "Your attachment was received and is being processed"
}
```

---

#### Happy Coding! If you encounter any issues or need help please don't hesitate to contact us!
