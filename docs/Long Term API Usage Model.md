

# Long Term API Usage Model

This purpose of this document is to help users understand:



1. The `_since` parameter, how it's processed by the API and why it’s important to use
2. The incremental export model
3. How to run first and second export jobs using `_since`

The goal of the AB2D API team is to provide best practices and usage strategies, which empower our users to efficiently use and maximize the value of our API.  This includes ensuring organizations understand how to access our production API and pull data frequently and incrementally.

The AB2D API team has provided and documented the following functionality to achieve this goal:

1. The ability to pull only the most recent claims using the [_since parameter](#the-since-parameter) including an
   [example](#run-first-export-job-v2).
1. The ability to detect claims status updates. See [Claims Representation Details](./Claims%20Representation%20Details.md) for more.
1. The ability to detect claims updates. See [Claims Representation Details](./Claims%20Representation%20Details.md) for more.

To help understand the `_since` parameter the following has also been included:

1. The underlying [model](#incremental-export-model-overview) behind exports.
1. A full [example](#example-export-workflow) demonstrating how to run multiple exports

<span style="text-decoration:underline;">IMPORTANT:</span> This document assumes that you have read and run your first production job already. If you have not, please read the 
[Usage Guide](https://github.com/CMSgov/ab2d-pdp-documentation/blob/main/docs/Production%20Access.md#usage-guide) in the Production Access document before proceeding.

## AB2D API Versions
AB2D supports both R4 and STU3 versions of the FHIR standard. [FHIR R4](http://hl7.org/fhir/R4/explanationofbenefit.html) 
is available using v2 of AB2D while [FHIR STU3](http://hl7.org/fhir/STU3/explanationofbenefit.html) can be accessed via AB2D v1. 
Both are available using the following URLs:

- R4- https://api.ab2d.cms.gov/api/v2/fhir
- STU3-https://api.ab2d.cms.gov/api/v1/fhir

Basic application functionality and API request processes are largely the same for both versions exception 
the way they process the `_since` parameter. These differences are outlined in the [`_since`](#the-since-parameter) section of this document below.

The data returned by each AB2D version are detailed within the [AB2D Data Dictionary](https://ab2d.cms.gov/data_dictionary.html). The differences in the data returned by each AB2D version are detailed within the [AB2D Migration Guide](https://github.com/CMSgov/ab2d-pdp-documentation/raw/main/AB2D%20STU3-R4%20Migration%20Guide%20Final.xlsx).

## The Since Parameter ##

Rather than pulling all the data available to the API on each call AB2D developed the `_since` parameter. 
The `_since` parameter allows users to grab EOB data added or updated since a specified date

Using the `_since` parameter reduces the time necessary to pull new data from the AB2D API by reducing 
the amount of duplicate data returned by the AB2D API. v1 and v2 of the interface deal with default
values differently.

Key aspects of the `_since` parameter

1. The earliest possible date to receive data is an organization's attestation date. The parameter will 
only work back to that date. If your organization attested prior to February 13th, 2020, please note:
    1. The parameter is only valid after February 13th, 2020 (2020-02-13T00:00:00.000-06:00). This functionality was 
   not implemented and will not work prior to this date. Any attempt to use a `_since` date before February 13th, 2020 will automatically be changed to February 13th, 2020.
3. The parameter must be provided as a zoned datetime following [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format: `yyyy-MM-dd'T'HH:mm:ss.SSSXXX`
4. A timezone must be provided with the zoned datetime or the export request will be rejected.
5. The datetime must not be in the future.

`_since` parameter differences between v1 (FHIR version STU3) and v2 (FHIR version R4):

**AB2D v2 (FHIR version R4)**
1. If no `_since` date is specified, the `_since` date will default to the last time a job was successfully run. Successfully run means that the job completed successfully, created all necessary files and those files were downloaded by the user. If the user has not yet run a successful job, the `_since` date will default to the attestation date or January 1, 2020, whichever is later.
2. If a _`since` date is specified, that will be used unless it’s before the attestation date or 2020-02-13T00:00:00.000-06:00 (the time `_since` was implemented)

**AB2D v1 (FHIR version STU3)**
1. A `_since` value must be added to each call in order to use the `_since` parameter. If no `_since` date is specified, the `_since` date will default to the attestation date or January 1, 2020, whichever is later.
2. If a `_since` date is specified, that will be used unless it’s before the attestation date or 2020-02-13T00:00:00.000-06:00 (the time `_since` was implemented)

### Valid and Invalid `_since` parameter values

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
   <td>The datetime is before the first February 13th, 2020 which is the earliest date that the `_since` parameter is valid
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


## Why the `_since` parameter in important
In this scenario, we have an organization called “XYZ”. Organization XYZ attests on March 1st, 2020. The organization 
runs its first job on November 1st, 2020 and subsequently decides to pull data once a month. The organization runs 
its second job on December 1st 2020.

### Without the `_since` parameter ###

1. The first job takes four hours to export all data available to them added between March 1st, 2020 and November 1st, 2020.
2. The second job takes four and a half hours to export all data available to them added between March 1st, 2020 and December 1st, 2020.

Without the `_since` parameter, the second job organization XYZ runs will pull all data from March 1st - November 1st 2020. 
The organization also pulls data from November 1st - December 1st 2020. Most of the data pulled is duplicate data. The only 
new data pulled is from the past month. Using the `_since` parameter prevents this.

### With the `_since` parameter ###
With the `_since` parameter equal to November 1st 2020 on the second job (either manually in v1/STU3 or by default in v2/R4):

1. The first job takes 4 hours to export all data available to them between March 1st, 2020 and November 1st 2020.
2. The second job takes half an hour to only export claims data available to them between November 1st, 2020 and December 1st, 2020.

With the `_since` parameter, the second job organization XYZ runs will pull data from November 1st - December 1st. Only 
pulling data for the past month will be quicker and smaller. Though a smaller number of claims may be pulled, a few may 
still be duplicates of already pulled claims.

## Incremental Export Model Overview

Below is a basic overview of the AB2D APIs designed usage model.

**For v2 (R4)**

In this example, different behaviour of default `_since` is defined
1. First job ever is started (JOB ID: ABC) with no `_since` date populated.
1. Once the job is complete, download the files
1. Wait until the next bi-weekly update (the recommended frequency to run jobs)
1. Start a job (JOB ID: DEF) with no `_since` parameter specified
1. Forget to download the files until after they’ve expired
1. Start a job (JOB ID: GHI) with no `_since` parameter specified - it will default to the create date of job ABC because job DEF was not completed successfully as no files were downloaded.
1. Once the job is complete, download the files
1. Repeat

In this usage model, AB2D API v2/R4 automatically populates `_since` value with the time of the last successfully completed job. 
ABC was the first job and returned all data since attestation. Job GHI had its `_since` date automatically populated to the date 
job ABC was created because the organization did not download the files from job DEF and thus the job was not considered completed 
successfully.

**For v1 (STU3)**

1. Start a job (JOB ID: ABC) and save the time the job started. The time ABC starts will be the `_since` parameter for your next job.
1. Wait until the next bi-weekly update (the recommended frequency to run jobs)
1. Start a job (JOB ID: DEF) manually entering the `_since` parameter with the time ABC started. The time DEF starts will be the `_since` parameter for your next job.
1. Repeat steps 2 & 3 indefinitely

In this usage model, an organization only pulls the last two weeks of claims data delivered to the AB2D API. Please note 
there may be duplicates returned even when using the `_since` parameter. The AB2D API errs on the side of potentially 
providing a claim twice rather than potentially never providing a claim.

## Example Export Workflow ##

This example demonstrates how to pull multiple times from the AB2D API using the Since-Datetime correctly for both 
v2 (R4) and v1 (STU3). Each command requires a valid Bearer token created by the following command:

   ```bash
   BASE64_AUTH=<base64 credentials>
   BEARER_TOKEN=$(curl -X POST  "$IDP_URL?grant_type=client_credentials&scope=clientCreds" \
          -H "Content-Type: application/x-www-form-urlencoded" \
          -H "Accept: application/json" \
          -H "Authorization: Basic ${BASE64_AUTH}" | jq --raw-output ".access_token")
   ```


### For v2 (R4) ###

#### Run First Export Job v2 ####

This example demonstrates how to pull multiple times from the AB2D API using the default `_since` capability

Running the job on December 16th (2020-12-16T00:00:00-05:00)

1. Retrieve a JWT token (see above)
2. Start the job without the `_since` parameter
   ```
   RESPONSE=$(curl "https://api.ab2d.cms.gov/api/v2/fhir/Patient/\$export?_outputFormat=application%2Ffhir%2Bndjson&_type=ExplanationOfBenefit \
   -sD - \
   -H "Accept: application/fhir+json" \
   -H "Prefer: respond-async" \
   -H "Authorization: Bearer ${BEARER_TOKEN}")
   ```
   
   Example response:

   ```
   echo $RESPONSE
   
   HTTP/1.1 202
   Content-Location: https://api.sandbox.ab2d.cms.gov/api/v2/fhir/Job/072e08ff-889e-4208-9318-20f9adddec17/$status
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

3. Repeatedly check status
4. Download files

**Run Second Export Job**

Running the job on December 30th (2020-12-16T00:00:00-08:00)

1. Retrieve a JWT token
2. Start the job no `_since` parameter

   ```
   RESPONSE=$(curl "https://api.sandbox.ab2d.cms.gov/api/v2/fhir/Patient/\$export?_outputFormat=application%2Ffhir%2Bndjson&_type=ExplanationOfBenefit \
   -sD - \
   -H "Accept: application/fhir+json" \
   -H "Prefer: respond-async" \
   -H "Authorization: Bearer ${BEARER_TOKEN}")
   ```

   Example response:

   ```
   echo $RESPONSE
   
   HTTP/1.1 202
   Content-Location: https://api.sandbox.ab2d.cms.gov/api/v2/fhir/Job/072e08ff-889e-4208-9318-20f9adddec17/$status
   # Save the _since datetime for the next job
   Since-Datetime: 2020-12-16T00:00:00-05:00
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
3. Repeatedly check status
4. Download files

### For v1 (STU3) ###

#### Run First Export Job v1

Running the job on December 16th (2020-12-16T00:00:00-08:00)

1. Retrieve  a JWT token (see above)
2. Start the job with the `_sinc`e parameter **and** save the `_since` parameter time for later use when processing the download files.

   ```bash
   RESPONSE=$(curl "https://api.ab2d.cms.gov/api/v1/fhir/Patient/\$export?_outputFormat=application%2Ffhir%2Bndjson&_type=ExplanationOfBenefit&_since=2020-12-01T00:00:00-08:00" \
        -sD - \
        -H "Accept: application/fhir+json" \
        -H "Prefer: respond-async" \
        -H "Authorization: Bearer ${BEARER_TOKEN}")
   ```

   Example response:
   ```bash
   echo $RESPONSE
   
    HTTP/1.1 202
    Content-Location: https://api.sandbox.ab2d.cms.gov/api/v1/fhir/Job/072e08ff-889e-4208-9318-20f9adddec17/$status
    # Save the _since datetime for the next job
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

**Run Second Export Job**

Running the job on December 30th (2020-12-16T00:00:00-08:00)

1. Retrieve a JWT token (see above)
2. Start the job with the `_since` parameter and save the `_since` parameter time for later use when processing the download files.

   Use the `Since-Datetime` from the previous job as the `_since` date for this next job.

   ```bash
   RESPONSE=$(curl "https://api.sandbox.ab2d.cms.gov/api/v1/fhir/Patient/\$export?_outputFormat=application%2Ffhir%2Bndjson&_type=ExplanationOfBenefit&_since=2020-12-16T00:00:00-08:00" \
        -sD - \
        -H "Accept: application/fhir+json" \
        -H "Prefer: respond-async" \
        -H "Authorization: Bearer ${BEARER_TOKEN}")
   ```
   
   Example response:
   ```bash
    HTTP/1.1 202 
    Content-Location: https://api.sandbox.ab2d.cms.gov/api/v1/fhir/Job/072e08ff-889e-4208-9318-20f9adddec17/$status
    # Save the _since datetime for the next job 
    Since-Datetime: 2020-12-14T21:00:00-08:00
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

