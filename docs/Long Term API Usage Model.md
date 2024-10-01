

# Long Term API Usage Model

This purpose of this document is to help users understand:



1. HTTP query parameters, how they're processed by the API, and why they're important to use
2. The incremental export model
3. How to run export jobs using `_since`

The goal of the AB2D API team is to provide best practices and usage strategies, which empower our users to efficiently use and maximize the value of our API.  This includes ensuring organizations understand how to access our production API and pull data frequently and incrementally.

The AB2D API team has provided and documented the following functionality to achieve this goal:

1. The ability to filter claims returned by date using [HTTP query parameters](#http-query-parameters).
2. The ability to run incremental exports. See [Incremental Export Model](#incremental-export-model-overview) for more. 
3. The ability to detect claims status updates. See [Claims Representation Details](./Claims%20Representation%20Details.md) for more.
4. The ability to detect claims updates. See [Claims Representation Details](./Claims%20Representation%20Details.md) for more.

<span style="text-decoration:underline;">IMPORTANT:</span> This document assumes that you have read and run your first production job already. If you have not, please read the 
[Usage Guide](https://github.com/CMSgov/ab2d-pdp-documentation/blob/main/docs/Production%20Access.md#usage-guide) in the Production Access document before proceeding.

## AB2D API Versions
This documentation is for AB2D API V2 ([FHIR R4](https://hl7.org/fhir/R4/)). This is the current and recommended version, which implements the [Bulk Data Access Implementation Guide V2.0.0](https://hl7.org/fhir/uv/bulkdata/). The `_until` parameter is only available with V2. For organizations using V1 ([FHIR STU3](https://hl7.org/fhir/STU3/)), visit our V1 documentation to learn about parameters. [Learn more about migrating from V1 to V2](https://github.com/CMSgov/ab2d-pdp-documentation/raw/main/AB2D%20STU3-R4%20Migration%20Guide%20Final.xlsx).

- V2 (R4) - api.ab2d.cms.gov/api/v2/fhir
- V1 (STU3) - api.ab2d.cms.gov/api/v1/fhir

## HTTP Query Parameters
The `_since` and `_until` parameters allow users to filter claims data returned by date, which reduces duplication and speeds up job times. The parameter values follow the [ISO datetime format](https://en.wikipedia.org/wiki/ISO_8601) (`yyyy-MM-dd'T'hh:mm:ss[+|-]hh:mm`). The time zone must be specified using + or - followed by `hh:mm`. There are currently 2 optional parameters, `_since` and `_until`, which can be used separately or together: 

Separately, these parameters allow users to pull data that has been updated since or until a specified date. You can use the [meta/lastUpdated](https://ab2d.cms.gov/data_dictionary.html#:~:text=The%20date%20that%20this%20record%20was%20last%20updated%20within%20the%20database.) property of each EOB resource to find when each record was last updated. This will help you compare claims data when using the  `_since` and `_until` parameters. 
- For `_since`, the earliest possible date is February 13th, 2020 (`2020-02-13T00:00:00-05:00`) or your organization's attestation date, whichever is later. If no `_since` date is specified, it will default to the datetime of your organization’s last successfully and fully downloaded job. If this is your first job, it will default to your earliest possible date. 
- For `_until`, the latest possible date is the current date. If no `_until` date is specified or you use a date from the future, it will default to the current date. 

Using the parameters together allows you to pull data that has been updated within a certain date range. However, the `_since` parameter value must be an earlier date than the `_until` parameter value. In other words, the `_until` datetime must have occurred after the `_since` datetime. 

## Examples of Parameter Values
<table>
  <tr>
   <td>Parameters
   </td>
   <td>Date Range of Export Requests
   </td>
  </tr>
  <tr>
   <td><code>_since</code> = null <br> 
      <code>_until</code> = null
   </td>
   <td>From <code>last successfully and fully downloaded job date</code><br>
to <code>current date</code>
   </td>
  </tr>
  <tr>
   <td><code>_since</code> = 1/1/2023 <br> 
      <code>_until</code> = null
   </td>
   <td>From <code>1/1/2023</code><br>
to <code>current date</code>
   </td>
  </tr>
  <tr>
   <td><code>_since</code> = null <br> 
      <code>_until</code> = 1/1/2024
   </td>
   <td>From <code>last successfully and fully downloaded job date</code><br>
to <code>1/1/2024</code>
   </td>
  </tr>
  </tr>
  <tr>
   <td><code>_since</code> = 1/1/2023 <br> 
      <code>_until</code> = 1/1/2024
   </td>
   <td>From <code>1/1/2023</code><br>
to <code>1/1/2024</code>
   </td>
  </tr>
</table>

### Valid and Invalid Parameter Values

<table>
  <tr>
   <td><code>_since</code> datetime
   </td>
   <td><code>_until</code> datetime
   </td>
   <td>Is it valid?
   </td>
   <td>Why?
   </td>
  </tr>
  <tr>
   <td><code>2020-10-10T00:00:00</code>
   </td>
   <td><code>2020-10-20T00:00:00</code>
   </td>
   <td>No
   </td>
   <td>A time zone must be provided with the datetime.
   </td>
  </tr>
  <tr>
   <td><code>2020-02-01T00:00:00+00:00</code>
   </td>
   <td><code>2020-02-28T00:00:00+00:00</code>
   </td>
   <td>No
   </td>
   <td>The <code>_since</code> datetime is before February 13th, 2020. The job will still run, but the datetime will be replaced with a different value.
   </td>
  </tr>
  <tr>
   <td><code>2020-10-10T16:00:00+00:00</code>
   </td>
   <td><code>3000-10-10T16:00:00+00:00</code>
   </td>
   <td>No
   </td>
   <td>The <code>_until</code> datetime is in the future. The job will still run, but the parameter value will be replaced with the current date.
   </td>
  </tr>
  <tr>
   <td><code>2020-10-10T09:00:00-08:00</code>
   </td>
   <td><code>2019-10-10T07:00:00-08:00</code>
   </td>
   <td>No
   </td>
   <td>The <code>_until</code> datetime is before the <code>_since</code> datetime. No data will be exported.
   </td>
  </tr>
   <tr>
   <td><code>2020-10-10T03:00:00-06:00</code>
   </td>
   <td><code>2021-10-10T06:00:00-06:00</code>
   </td>
   <td>Yes
   </td>
   <td>The <code>_until</code> datetime is after the <code>_since</code> datetime. The datetimes are valid and follow the ISO format with time zones.
   </td>
  </tr>
</table>


## Parameter Scenario
There may be use cases where a specific date range of EOBs is required. For example:
1. Your organization decides to incrementally export data and runs a job on December 1st, 2023 for a month’s worth of data. 
2. The job takes 4 hours to export all the data available to you updated between November 1st, 2023 and December 1st, 2023. 
3. Your organization realizes 1 week’s worth of data was corrupted in your database. 

Instead of rerunning the entire job, which would take 4 hours and result in duplicate data, you decide to use the `_since` and `_until` parameters to target the corrupted data:

4. Your organization identifies the corrupted data as claims last updated between November 12-18, 2023. 
5. Your organization runs a second job and uses November 12, 2023 as the `_since` parameter value and November 18, 2023 as the `_until` parameter value. 
6. The job takes less than 1 hour to export the week’s worth of missing data from your database. 

Using `_since` and `_until` to specify the date range of your export request speeds up job times as your organization doesn’t need to redownload unnecessary data. 

## Incremental Export Model Overview
One recommended way to use the AB2D API is to periodically export any data that has been updated since your last job. We encourage bi-weekly exports to stay updated on new claims data.

Incremental exports occur by default when you start a job, even without using parameters, so no further action is required.  This is because `_since` defaults to the datetime of your last successful export, and `_until` defaults to the current datetime. 

**Example Scenario**

This is a scenario demonstrating how the default capability works for `_since` and `_until`:
1. You start your first job (JOB ID: ABC).
2. Once the job is complete, you download the files.
3. You wait until the next bi-weekly update (the recommended frequency to run jobs).
4. You start a new job (JOB ID: DEF).
5. You forget to download the files and they’ve expired after 72 hours.
6. You start another job (JOB ID: GHI). The `_since` value will default to when you started job ABC, not job DEF. This is because no files were downloaded with job DEF, so it was not completed successfully.
7. Once the job is complete, download the files.
8. Repeat.
   
In this usage model, AB2D automatically populates the `_since` value with the datetime of your last successfully completed export. For Job GHI, this is the date job ABC was created. Job DEF isn’t considered a successfully completed export since its files weren’t downloaded.

## Example Export Workflow Using Parameters
Visit [Production Access](https://github.com/CMSgov/ab2d-pdp-documentation/blob/main/docs/Production%20Access.md) to learn more about the export workflow for the AB2D API. Query parameters can be used while starting an export job. 

The following examples use `_since` and `_until` query parameters separately and together. Note that ISO8601 dates include characters that can’t appear in URLs. [Learn more about percent-encoding](https://en.wikipedia.org/wiki/Percent-encoding). There are unencoded and percent-encoded examples. Only the encoded versions will work, but the unencoded versions will show how the URL is formed before encoding.

**Only `_since` Parameter Is Specified**
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




**Only `_until` Parameter Is Specified**
<table>
  <tr>
   <td>Unencoded
   </td>
   <td>Percent-encoded
   </td>
  </tr>
  <tr>
   <td>https://sandbox.ab2d.cms.gov/api/v2/fhir/Patient/$export?_until=2024-06-15T00:00:00.000-05:00
   </td>
   <td>https://sandbox.ab2d.cms.gov/api/v2/fhir/Patient/$export?_until%3D2024-06-15T00%3A00%3A00.000-05%3A00
   </td>
  </tr>
</table>

curl command 

 ```
curl "https://api.ab2d.cms.gov/api/v2/fhir/Patient/\$export?_until%3D2024-06-15T00%3A00%3A00.000-05%3A00" \-H "accept: application/json" \
-H "Accept: application/fhir+json" \
-H "Prefer: respond-async" \
-H "Authorization: Bearer ${BEARER_TOKEN}
   ```



**Both `_since` and `_until` Parameters Are Specified**
<table>
  <tr>
   <td>Unencoded
   </td>
   <td>Percent-encoded
   </td>
  </tr>
  <tr>
   <td>https://sandbox.ab2d.cms.gov/api/v2/fhir/Patient/$export?_since=2023-06-15T00:00:00.000-05:00&_until=2024-06-15T00:00:00.000-05:00
   </td>
   <td>https://sandbox.ab2d.cms.gov/api/v2/fhir/Patient/$export?_since%3D2023-06-15T00%3A00%3A00.000-05%3A00%26_until%3D2024-06-15T00%3A00%3A00.000-05%3A00
   </td>
  </tr>
</table>

curl command 

 ```
curl "https://api.ab2d.cms.gov/api/v2/fhir/Patient/\$export?_since%3D2023-06-15T00%3A00%3A00.000-05%3A00%26_until%3D2024-06-15T00%3A00%3A00.000-05%3A00" \-H "accept: application/json" \
-H "Accept: application/fhir+json" \
-H "Prefer: respond-async" \
-H "Authorization: Bearer ${BEARER_TOKEN}
   ```
