

# Long Term API Usage Model – V1

This purpose of this document is to help users understand:



1. The `_since` parameter, how it's processed by the API, and why it’s important to use
2. The incremental export model
3. How to run first and second export jobs using `_since`

The goal of the AB2D API team is to provide best practices and usage strategies, which empower our users to efficiently use and maximize the value of our API.  This includes ensuring organizations understand how to access our production API and pull data frequently and incrementally.

The AB2D API team has provided and documented the following functionality to achieve this goal:

1. The ability to pull only the most recent claims using the [_since parameter](#the-since-parameter).
1. The ability to detect claims status updates. See [Claims Representation Details](./Claims%20Representation%20Details.md) for more.
1. The ability to detect claims updates. See [Claims Representation Details](./Claims%20Representation%20Details.md) for more.

To help understand the `_since` parameter the following has also been included:

1. The underlying [model](#incremental-export-model-overview) behind exports.
1. A full [example](#example-export-workflow) demonstrating how to run multiple exports

<span style="text-decoration:underline;">IMPORTANT:</span> This document assumes that you have read and run your first production job already. If you have not, please read the 
[Usage Guide](https://github.com/CMSgov/ab2d-pdp-documentation/blob/main/docs/Production%20Access.md#usage-guide) in the Production Access document before proceeding.

## AB2D API Versions
This documentation is for AB2D API V1 ([FHIR STU3](https://hl7.org/fhir/STU3/)). This is the older version, which implements the [Bulk Data Access Implementation Guide V1.0.1](https://hl7.org/fhir/uv/bulkdata/STU1.0.1/). For organizations using V2 ([FHIR R4](https://hl7.org/fhir/R4/)) as recommended, visit our [V2 documentation](https://github.com/CMSgov/ab2d-pdp-documentation/blob/main/docs/Long%20Term%20API%20Usage%20Model.md) to learn about parameters. The `_until` parameter is only available with V2.  [Learn more about migrating from V1 to V2](https://github.com/CMSgov/ab2d-pdp-documentation/raw/main/AB2D%20STU3-R4%20Migration%20Guide%20Final.xlsx). 


- V2 (R4) - api.ab2d.cms.gov/api/v2/fhir
- V1 (STU3) - api.ab2d.cms.gov/api/v1/fhir

## The Since Parameter ##

The `_since` parameter allows users to filter for claims data last updated since a specified date. This reduces duplication and speeds up job times. It follows the ISO datetime format (`yyyy-MM-dd'T'hh:mm:ss[+|-]hh:mm`). The time zone must be specified using + or - followed by `hh:mm`. 

The earliest possible date is February 13th, 2020 (`2020-02-13T00:00:00-05:00`) or your organization's attestation date, whichever is later. If no `_since` date is specified, it will default to your earliest possible date. It is highly recommended and considered best practice to use the `_since` parameter on V1 API calls. 

The `_until` parameter filters for claims data last updated after a specified date. This parameter is only available with V2 ([FHIR R4](https://hl7.org/fhir/R4/)), the current and recommended version of the API. 

### Valid and Invalid Parameter Values

<table>
  <tr>
   <td><code>_since</code>
   </td>
   <td>Is it valid?
   </td>
   <td>Why?
   </td>
  </tr>
  <tr>
   <td><code>2020-02-01T00:00:00.000+00:00</code>
   </td>
   <td>No
   </td>
   <td>The datetime is before February 13th, 2020. The job will still run, but the value will be replaced with your earliest possible date. 
   </td>
  </tr>
  <tr>
   <td><code>2020-10-10T00:00:00.000</code>
   </td>
   <td>No
   </td>
   <td>A time zone must be provided with the datetime.
   </td>
  </tr>
  <tr>
   <td><code>2020-10-10T16:00:00.000+10:00</code>
   </td>
   <td>Yes
   </td>
   <td>The datetime is after February 13th, 2020 and follows the ISO format.
   </td>
  </tr>
  <tr>
   <td><code>2020-10-10T16:00:00.000-10:00</code>
   </td>
   <td>Yes
   </td>
   <td>The datetime is after February 13th, 2020 and follows the ISO format.
   </td>
  </tr>
</table>


## Why the `_since` Parameter Is Important
In this scenario, your organization attests on March 1st, 2020. Your organization runs its first job on November 1st, 2020 and subsequently decides to pull data once a month. Your organization runs its second job on December 1st, 2020.

### Without the `_since` Parameter ###

1. The first job takes 4 hours to export all data available to you added between March 1st, 2020 and November 1st, 2020.
2. The second job takes 4 1/2 hours to export all data available to you added between March 1st, 2020 and December 1st, 2020.

Without the `_since` parameter, the second job will pull all data between March 1st, 2020 and November 1st, 2020. Most of the data pulled is duplicate data. The only 
new data pulled is from the past month (November 1st, 2020 to December 1st, 2020). Using the `_since` parameter can help prevent duplicate data and speed up job times.

### With the `_since` Parameter ###
With the `_since` parameter value manually set to November 1st, 2020 on the second job:

1. The first job takes 4 hours to export all data available to you between March 1st, 2020 and November 1st, 2020.
2. The second job takes 1/2 hour to only export claims data available to you between November 1st, 2020 and December 1st, 2020.

With the `_since` parameter, the second job will only pull data from November 1st, 2020 to December 1st, 2020. Only 
pulling data for the past month will be quicker and smaller. Though a smaller number of claims may be pulled, a few may 
still be duplicates of already pulled claims.

## Incremental Export Model Overview

One recommended way to use AB2D is to periodically export any data that has been updated since your last job. We encourage bi-weekly exports to stay updated on new claims data. By using the `_since` parameter, you can filter for claims data last updated since your last job export. This way, you can run manual incremental exports. 

**Example Scenario**
1. Start a job (JOB ID: ABC) and save the datetime the job started. The datetime ABC starts will be the `_since` parameter value for your next job.
2. Wait until the next bi-weekly update (the recommended frequency to run jobs).
3. Start a job (JOB ID: DEF), manually entering the datetime from step 1 as the `_since` parameter value. The datetime DEF starts will then be the `_since` parameter value for your next job.
4. Repeat steps 2 & 3 indefinitely.

In this usage model, an organization only pulls the last 2 weeks of claims data delivered to the AB2D API. Please note there may be duplicates returned even when using the `_since` parameter. AB2D errs on the side of potentially providing a claim twice rather than potentially never providing a claim.

## Example Export Workflow ##
Visit [Production Access](https://github.com/CMSgov/ab2d-pdp-documentation/blob/main/docs/Production%20Access.md#expected-workflow) to learn more about the export workflow for the AB2D API. The `_since` parameter can be used while starting an export job. 

There are both unencoded and percent-encoded examples of the `_since` parameter. Note that ISO8601 dates include characters that can’t appear in URLs. [Learn more about percent-encoding](https://en.wikipedia.org/wiki/Percent-encoding). Only the encoded version will work, but the unencoded version will show how the URL is formed before encoding.

**`_since` Parameter Is Specified**
<table>
  <tr>
   <td>Unencoded
   </td>
   <td>Percent-encoded
   </td>
  </tr>
  <tr>
   <td>https://sandbox.ab2d.cms.gov/api/v2/fhir/Patient/$export?_since=2023-02-13T00:00:00.000-05:00
   </td>
   <td>https://sandbox.ab2d.cms.gov/api/v2/fhir/Patient/$export?_since%3D2023-02-13T00%3A00%3A00.000-05%3A00
   </td>
  </tr>
</table>

curl command 

 ```
   curl "https://api.ab2d.cms.gov/api/v2/fhir/Patient/\$export?_since%3D2023-02-13T00%3A00%3A00.000-05%3A00" \
-H "Accept: application/json" \
-H "Accept: application/fhir+json" \
-H "Prefer: respond-async" \
-H "Authorization: Bearer ${BEARER_TOKEN}
   ```
