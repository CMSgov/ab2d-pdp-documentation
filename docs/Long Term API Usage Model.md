

# Long Term API Usage Model

This purpose of this document is to help users understand:



1. The _since parameter, how its processed by the API and why it’s important to use
2. The incremental export model
3. How to run first and second export jobs using _since

The goal of the AB2D API team is to provide best practices and usage strategies, which empower our users to efficiently use and maximize the value of our API.  This includes ensuring organizations understand how to access our production API and pull data frequently and incrementally.

The AB2D API team has provided and documented the following functionality to achieve this goal:



1. The ability to pull only the most recent claims using the [_since parameter](#the-since-parameter) including an
   [example](#why-you-should-use-the-_since-parameter).
1. The ability to detect claims status updates. See [Claims Representation Details](./Claims%20Representation%20Details.md) for more.
1. The ability to detect claims updates. See [Claims Representation Defails](./Claims%20Representation%20Details.md) for more.

To help understand the _since parameter the following has also been included:

1. The underlying [model](#incremental-export-model-overview) behind exports.
1. A full [example](#example-export-workflow) demonstrating how to run multiple exports

<span style="text-decoration:underline;">IMPORTANT:</span> This document assumes that you have read and run your first production job already. If you have not, please read the [Usage Guide](./docs/Production%20Access.md#usage-guide) in the Production Access document before proceeding.


## The Since Parameter

Rather than pulling all the data available to the API on each call AB2D developed the _since parameter. The _since parameter allows users to grab EOB data added or updated since a specified date

Using the _since parameter reduces the time necessary to pull new data from the AB2D API by reducing the amount of duplicate data returned by the AB2D API.

Internally, the AB2D API rounds down the _since parameter to the previous Tuesday at midnight Eastern Standard Timezone (EST). The reasoning and examples of this are discussed extensively in the sections below.

Key aspects of the _since parameter



1. The earliest possible date to receive data is an organization's attestation date. The parameter will only work up to that date. If your organization attested prior to February 13th, 2020, please note:
    1.  The parameter is only valid after February 13th, 2020 (2020-02-13T00:00:00.000-06:00). This functionality was not implemented and will not work prior to this date. Any attempt to use a since date before February 13th, 2020 will automatically be changed to February 13th, 2020.
2. The parameter must be provided as a zoned datetime following [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format: _yyyy-MM-dd'T'HH:mm:ss.SSSXXX_
3. A timezone must be provided with the zoned datetime or the export request will be rejected.
4. The datetime must not be in the future.


### Valid and Invalid _since parameter values


<table>
  <tr>
   <td>_since
   </td>
   <td>Is it valid?
   </td>
   <td>Why?
   </td>
  </tr>
  <tr>
   <td>2020-02-01T00:00:00.000+00:00
   </td>
   <td>No
   </td>
   <td>The datetime is before the first February 13th, 2020 which is the earliest date that the since parameter is valid
   </td>
  </tr>
  <tr>
   <td>2020-10-10T00:00:00.000
   </td>
   <td>No
   </td>
   <td>A timezone must be provided with the datetime
   </td>
  </tr>
  <tr>
   <td>2020-10-10T16:00:00.000+10:00
   </td>
   <td>Yes
   </td>
   <td>The datetime is after February 13th, 2020 and follows the ISO format
   </td>
  </tr>
  <tr>
   <td>2020-10-10T16:00:00.000-10:00
   </td>
   <td>Yes
   </td>
   <td>The datetime is after February 13th, 2020 and follows the ISO format
   </td>
  </tr>
</table>



### How the API processes the _since parameter

**The AB2D API rounds down the _since parameter to the previous Tuesday at Midnight** Eastern Standard Timezone (EST).

AB2D claims data source receives weekly updates from its upstream data source. The update process takes some time and may interfere with the _since date causing an organization to miss claims data.

To prevent this scenario, the AB2D API rounds down the _since parameter to a date that we know an update will not be running.


### Examples


For reference, December 1st, 2020 (2020-12-01) was a Tuesday.


<table>
  <tr>
   <td>Since
   </td>
   <td>Internal Datetime (EST)
   </td>
   <td>Returned Since-Datetime
   </td>
  </tr>
  <tr>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
  </tr>
  <tr>
   <td>2020-12-01T00:00:00.000-08:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
   <td>2020-11-30T21:00:00.000-08:00
   </td>
  </tr>
  <tr>
   <td>2020-11-30T21:00:00.000-08:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
   <td>2020-11-30T21:00:00.000-08:00
   </td>
  </tr>
  <tr>
   <td>2020-12-02T00:00:00.000-05:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
  </tr>
  <tr>
   <td>2020-12-02T00:00:00.000+05:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
   <td>2020-12-01T10:00:00.000+05:00
   </td>
  </tr>
  <tr>
   <td>2020-12-03T00:00:00.000-08:00
   </td>
   <td>2020-12-01T00:00:00.000-05:00
   </td>
   <td>2020-11-30T21:00:00.000-08:00
   </td>
  </tr>
</table>



## Why you should use the _since parameter

In this scenario, we have an organization called “XYZ”. Organization XYZ attests on March 1st 2020. The organization runs their first job on November 1st 2020 and decides to pull data once a month. The organization runs their second job on December 1st 2020.


### If XYZ does not use the _since parameter



1. The first job takes four hours to export all data available to them added between March 1st, 2020 and November 1st 2020.
2. The second job takes four and a half hours to export all data available to them added between March 1st 2020 and December 1st 2020.

Without the _since parameter, the second job organization XYZ runs will pull all data from March 1st - November 1st 2020. The organization also pulls data from November 1st - December 1st 2020. Most of the data pulled is duplicate data. The only new data pulled is from the past month. Using the _since parameter prevents this.


### If XYZ uses the _since parameter

Organization XYZ sets `_since` equal to November 1st 2020 on their second job



1. The first job takes 4 hours to export all data available to them between March 1st, 2020 and November 1st 2020.
2. The second job takes half an hour to only export claims data available to them between November 1st, 2020 and December 1st 2020.

With the _since parameter, the second job organization XYZ runs will pull data from November 1st - December 1st. Only pulling data for the past month will be quicker and smaller. Though a smaller number of claims may be pulled, a few may still be duplicates of already pulled claims.


## Incremental Export Model Overview

Below is a basic overview of the AB2D APIs designed usage model.



1. Start a job (ABC)  and save the time the job started. The time ABC starts will be the _since parameter for your next job.
2. Wait until the next bi-weekly update (the recommended frequency to run jobs)
3. Start a job (DEF) with the _since parameter as the time ABC started. The time DEF starts will be the _since parameter for your next job.
4. Repeat steps 2 & 3 indefinitely

In this usage model, an organization only pulls the last two weeks of claims data delivered to the AB2D API. Please note there may be duplicates returned even when using the _since parameter. The AB2D API errs on the side of potentially providing a claim twice rather than potentially never providing a claim.


## Example Export Workflow

This example demonstrates how to pull multiple times from the AB2D API using the Since-Datetime correctly.


### Run First Export Job

Running the job on December 16th (2020-12-16T00:00:00-08:00)



1. Retrieve a JWT token

   ```bash
   BASE64_AUTH=<base64 credentials>
   BEARER_TOKEN=$(curl -X POST  "$IDP_URL?grant_type=client_credentials&scope=clientCreds" \
          -H "Content-Type: application/x-www-form-urlencoded" \
          -H "Accept: application/json" \
          -H "Authorization: Basic ${BASE64_AUTH}" | jq --raw-output ".access_token")
   ```

2. Start the job with the since parameter and** **save the rounded since parameter time for later use when processing the download files.

   ```bash
   RESPONSE=$(curl "https://api.ab2d.cms.gov/api/v1/fhir/Patient/\$export?_outputFormat=application%2Ffhir%2Bndjson&_type=ExplanationOfBenefit&_since=2020-12-01T00:00:00-08:00" \
        -sD - \
        -H "accept: application/json" \
        -H "Accept: application/fhir+json" \
        -H "Prefer: respond-async" \
        -H "Authorization: Bearer ${BEARER_TOKEN}")
   ```

   Example response:
   ```bash
    HTTP/1.1 202
    Content-Location: https://api.sandbox.ab2d.cms.gov/api/v1/fhir/Job/072e08ff-889e-4208-9318-20f9adddec17/$status
    Since-Datetime: 2020-11-30T21:00:00-08:00
    X-Content-Type-Options: nosniff
    X-XSS-Protection: 1; mode=block
    Cache-Control: no-cache, no-store, max-age=0, must-revalidate
    Pragma: no-cache
    Expires: 0
    Strict-Transport-Security: max-age=31536000 ; includeSubDomains
    X-Frame-Options: DENY
    Content-Length: 0
    Date: Mon, 01 Feb 2021 17:33:56 GMT
   ```

3. Save the date the job was started on. December 16th (2020-12-16T00:00:00-08:00).
4. Repeatedly check status
5. Download files


### Run Second Export Job

Running the job on December 30th (2020-12-16T00:00:00-08:00)



1. Retrieve a JWT token

   ```bash
   BASE64_AUTH=<base64 credentials>
   BEARER_TOKEN=$(curl -X POST  "$IDP_URL?grant_type=client_credentials&scope=clientCreds" \
          -H "Content-Type: application/x-www-form-urlencoded" \
          -H "Accept: application/json" \
          -H "Authorization: Basic ${BASE64_AUTH}" | jq --raw-output ".access_token")
   ```

2. Start the job with the since parameter and **save the rounded since parameter time for later use when processing the download files.**

   ```bash
   RESPONSE=$(curl "https://api.sandbox.ab2d.cms.gov/api/v1/fhir/Patient/\$export?_outputFormat=application%2Ffhir%2Bndjson&_type=ExplanationOfBenefit&_since=2020-12-16T00:00:00-08:00" \
        -sD - \
        -H "accept: application/json" \
        -H "Accept: application/fhir+json" \
        -H "Prefer: respond-async" \
        -H "Authorization: Bearer ${BEARER_TOKEN}")
   ```
   
   Example response:
   ```bash
    HTTP/1.1 202 
    Content-Location: https://api.sandbox.ab2d.cms.gov/api/v1/fhir/Job/072e08ff-889e-4208-9318-20f9adddec17/$status
    Since-Datetime: 2020-12-14T21:00:00-08:00 # Save since datetime for later
    X-Content-Type-Options: nosniff
    X-XSS-Protection: 1; mode=block
    Cache-Control: no-cache, no-store, max-age=0, must-revalidate
    Pragma: no-cache
    Expires: 0
    Strict-Transport-Security: max-age=31536000 ; includeSubDomains
    X-Frame-Options: DENY
    Content-Length: 0
    Date: Mon, 01 Feb 2021 17:33:56 GMT
   ```

3. Save the date the job was started on. December 31st (2020-12-31T00:00:00-08:00).
4. Repeatedly check status
5. Download files