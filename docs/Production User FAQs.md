# General Definitions In Use

## Authentication
### Base64 
A basic encoding meant to take your token from a text format to a standard ASCII format that is easier for computers to interpret.

### Okta & Identity Providers
An Identity Provider (IdP) is a service that stores, verifies and manages user identities. CMS uses a third party identity provider tool named [Okta](https://support.okta.com/help/s/article/What-is-Okta?language=en_US) for organizations using the AB2D API. The AB2D Team will provide each participating client organization with a set of Okta credentials required to retrieve a one-time use, time boxed JSON web token (JWT)/Bearer token used to access the AB2D API.

### JSON Web Token (JWT) & Bearer Token
(These may be used interchangeably).

JSON web tokens (JWT), also referred to as Bearer Tokens,  are used during the [OAuth 2.0](https://oauth.net/2/) authentication/authorization process to verify a user's identity. During this process, a user (in this case an PDP organization) will provide Okta with the credentials provided by the AB2D Team. Okta will return the user a JWT which serves as verification of your identity with the AB2D API.  JWTs are temporary and expire after an hour, after which time you must retrieve a new token.

A user must provide a valid JWT everytime they wish to access the AB2D API. AB2D checks with Okta to ensure the token provided matches their records to verify your identity and authorization. If the requirements are satisfied, then access is granted to the AB2D API.

For more information see https://jwt.io/introduction/. 

## AB2D API & Job Quick Overview
The AB2D API is a CMS interface that standalone Part D Providers can use to export Medicare A & B parts claims data.

### Definitions
- Application Programming Interface (API): publicly facing interface used to share information that organizations can access if they have completed the on-boarding process with the AB2D Team.
- Job: a unit of work exporting Parts A & B information and compiling it for an organization to download. Jobs are triggered by the BulkDownloadAPI.

### List of APIs that AB2D Provides
- Export API: start a job to export data 
- Status API: monitor the status of a job
- Download API: download the results of a job
- Health API: check the health of the AB2D API (up, down, busy, etc.)

### HTTP Headers
HTTP headers contain additional information about an HTTP request or response that can be used to do additional filtering/processing.

List of headers that organizations provide on their requests:
- Authorization: the JSON web token secret retrieved from Okta and used to authenticate the organization
- Accept: The type of response the organization is expecting to receive (JSON, binary, text, etc.) from the AB2D API
- Prefer: what kind of response we want to receive (synchronous vs. asynchronous)

List of headers that AB2D provides as responses:
- Content-Location: HTTPS URL to get the status of a job or a file
- Content-Length: number of bytes in the response body
- Content-Type: The type of content contained in the response (should match the Accept header)
- X-Progress: current progress of a job returned by status API

### HTTP Codes
- 200 -  Request completed successfully
- 202 - Request accepted but still processing
- 400 - Bad Request, general response when something with the request is blatantly wrong. Like missing a request parameter or body.
- 401 - Unauthorized, authentication has not been done or has not succeeded.
- 403 - Forbidden, authentication succeeded but the user doesn't have permissions to access the requested data. May also be returned when authentication fails.
- 404 - Resource or page not found. You could authenticate but the API endpoint does not exist.
- 405 - Method not allowed. You are trying to execute a method which the endpoint does not support. Example calling /secure/get as a POST method but the endpoint actually only supports GET methods.
- 429 - Too Many Requests. You are trying to create a Job when the max number of Jobs have been created.

More information on standard HTTP Codes: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes 

## Common User Questions
- **What is a Bearer Token?** <br>A bearer token is just another name for a JSON Web Token (JWT).
- **How do I get the Bearer Token?** <br>Retrieve a JSON Web Token from the identity provider, Okta, using your credentials.
- **What does a 403 error mean when trying to get the token?**
    1. The credentials provided may have a typo or syntax error including spaces, hidden characters, etc.
    1. The credentials provided may have been encoded to base64 incorrectly. Check that the base64 credentials are correct.
    1. Check that you are connecting to the production identity provider (idp.cms.gov) and not the sandbox identity provider (test.idp.idm.cms.gov).
    1. Check that you are providing the correct Authorization header (the header should be “Authorization : Basic Auth”).
- **How do I figure out what my IP address is?**
    1. If you have a browser, open it on the machine you wish to connect to the AB2D API from, and go to this website http://checkip.amazonaws.com/ 
    1. If you do not have a browser query from the command line of the machine you wish to connect to the AB2D API from:
       1. On Linux/Mac open a terminal and run the following command: “curl -X GET checkip.amazonaws.com”
       1. On Windows open a Powershell terminal and run the following command “Invoke-RestMethod -Method GET checkip.amazonaws.com”
- **The IP address is not correct, what should I do?**
    1. Determine if the machine in use is correct
    1. If the machine is correct then check with your IT team that the IP address is a static ip address. If the IP address is not static then it may have changed. We register IP specific IP addresses that are allowed to connect with the AB2D API, so you must have a static ip address.
- **How do I determine if my address can connect to AB2D? Using one of three methods:**
    1. On the command line of the machine you wish to connect to the AB2D API, run one of the two following commands:
        1. On Linux/Mac in a terminal run: “curl -X GET https://api.ab2d.cms.gov/health --verbose”
        1. On Windows in a powershell terminal run “Invoke-RestMethod -Method GET https://api.ab2d.cms.gov/health”
    1. In Postman type create a new GET request against the following URL: https://api.ab2d.cms.gov/health. If the response has an HTTP status of 200 then the request is okay.
    1. In a browser, on the machine, go to the following URL https://api.ab2d.cms.gov/swagger-ui/index.html 
- **If I attempt to start a job, and I get __ error code, what do I do?**
    1. 403 - Then your bearer token may not be correct or may have expired, regenerate the bearer token.
    2. 400 or 404 - Then the endpoint you attempted to start a job at does not exist. Check that you are using the correct endpoint.
    3. 429 - You have requested too many jobs. View the Content-Location Header to get the status of a job or a file.
- **What does a 200 code mean when starting a job?** <br>This means the export job has started successfully. In the headers in the request should be the endpoint to find out the current status of the job.
- **What does a 202 code mean when querying the status?** <br>This means the export job is currently running and has not finished. In the headers returned in the response you can see what % done the job is.
- **Why is my job stuck at 0% complete? Why is my job queueing and not starting immediately?**<br>
The AB2D API can only run so many jobs at once. If the API is busy the job will be queued in a backlog to be executed at some point in the future.
- **How do I cancel a job that has already started?**
    1. On the command line run one of the two following commands:
        1. On Linux/Mac in a terminal run: “curl -X DELETE https://api.ab2d.cms.gov/api/v1/fhir/Job/{jobUuid}/$status --verbose”. Make sure to fill in {jobUuid} with the id of the running job.
        1. On Windows in a powershell terminal run “Invoke-RestMethod -Method DELETE https://api.ab2d.cms.gov/api/v1/fhir/Job/{jobUuid}/$status”. Make sure to fill in {jobUuid} with the id of the running job.
    1. In Postman type create a new DELETE request against the following URL: https://api.ab2d.cms.gov/api/v1/fhir/Job/{jobUuid}/$status. Note the request parameter jobUuid which must be set in Postman.
    1. In a browser, on the machine, go to the following URL https://api.ab2d.cms.gov/swagger-ui/index.html#/Status/deleteRequestUsingDELETE and enter the job id.
- **What response am I expecting when the job finishes successfully?** <br>You should expect to get a response with a JSON object containing the list of files you can download and the URLs to query to do so.
- **What happens if the job fails?** <br>The status API will return an error concerning the job. Any files created by the job are deleted.
- **Can I retrieve the results of a job that fails?** <br>No you cannot.
- **I tried to download the files and got an error message**
    1. 403 - your bearer token may have expired or may be incorrect, regenerate the bearer token.
    2. 4xx - the files are deleted 72 hours after a job completes or if the files have been downloaded more than six times.
- **What is FHIR STU3 and R4?** <br>STU3 and R4 are different FHIR versions for data transmission. AB2D provides STU3 and R4 versions of one type of FHIR
object, ExplanationOfBenefit. A detailed explanation of these versions are defined
[here for R4](http://hl7.org/fhir/R4/explanationofbenefit.html) and [here for STU3](http://hl7.org/fhir/STU3/explanationofbenefit.html). 

- **How does AB2D provide FHIR versions to users?**<br>AB2D supports both FHIR STU3 and R4 using different 
  production versions of the API. AB2D v1 = STU3 and AB2D v2 = R4. They are available using the following URLs :
  - R4 [https://api.ab2d.cms.gov/api/v2/fhir](https://api.ab2d.cms.gov/api/v2/fhir)
  - STU3 [https://api.ab2d.cms.gov/api/v2/fhir](https://api.ab2d.cms.gov/api/v1/fhir)

Requests made to both versions of the API are largely the same except how they process the
  `_since` parameter. 

The data returned by each AB2D version are detailed within the [AB2D Data Dictionary](https://ab2d.cms.gov/data_dictionary.html). The differences in the data returned by each AB2D version are detailed within the [AB2D Migration Guide](https://github.com/CMSgov/ab2d-pdp-documentation/AB2D%20STU3-R4%20Migration%20Guide%20Final.xlsx).
- **What is default `_since`?**<br>
The `_since` parameter is used to limit the returned data from the AB2D job. The `_since` date provided filters claims that were made 
available after the specified date (as defined in the claims update date). The purpose of the `_since` parameter is to allow users to 
pull incremental data dumps reducing data duplication and speeding up job times. For a more detailed explanation, including examples, 
please reference our [Production Access and Long Term Usage](https://github.com/CMSgov/ab2d-pdp-documentation/blob/main/docs/Long%20Term%20API%20Usage%20Model.md) documentation in GitHub. 
- **What is the difference between different versions of the AB2D API?**</a><br>
As referenced above, the `_since` parameter has different default behavior in AB2D v1/STU3 vs v2/R4. In v1 of our interface (STU3), a value 
must be added to each call in order to use the `_since` parameter. if no `_since` value is specified, the value defaults to a contract's 
attestation date. In v2 of our interface (R4), if no `_since` value is specified, the value defaults to the creation time of the 
last successful job (both completed successfully and all files downloaded). This allows for automatic incremental data dumps 
without having to remember the time of the last job. If this is the first job performed by the user,
the `_since` date for either version defaults to the attestation date if none is provided. If the user specifies 
a `_since` date, this date will be used.

