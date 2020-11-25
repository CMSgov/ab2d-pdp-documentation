# Welcome to the AB2D Documentation repo

In this repository you will find documentation related to accessing production and retrieving real claims data using the AB2D (Claims Data to Part D Sponsors) API. Please read the documentation thoroughly as small variations such as a missing space or missing quote can prevent the the scripts from running properly. When using production, the clients have the ability to download PII/PHI information. You should therefore ensure the environment in which these scripts are run is secured in a way to allow for storage of PII/PHI. Additionally, when used in the production environment the scripts will require use of your production credentials. As such, please ensure that your credentials are handled in a secure manner and not printed to logs or the terminal. Ensuring the privacy of data is the responsibility of each user and/or organization.

## Table of Contents

* [AB2D Documentation Repo](https://github.com/CMSgov/ab2d-private-)
  * [Accessing Production](https://github.com/CMSgov/ab2d-private-/docs/proddocs.md)

## Sample Client Repos

The sample client repos may be referenced in this documentation can be found below:
* [Python Sample Client](https://github.com/CMSgov/ab2d-sample-client-python/)
* [Bash Sample Client](https://github.com/CMSgov/ab2d-sample-client-bash/)
* [Powershell Sample Client](https://github.com/CMSgov/ab2d-sample-client-powershell/)

It is important to note that the AB2D team does **not** regularly maintain the sample clients. Additionally, a best-effort was made to ensure the clients are secure but they have **not** undergone comprehensive formal security testing. Each user/organization is responsible for conducting their own review and testing prior to implementation

## Asking for Help

If issues persist using these or other scripts email the AB2D team at [ab2d@semanticbits.com](mailto:ab2d@semanticbits.com)
or join our Google Groups at:
[https://groups.google.com/u/1/g/cms-ab2d-api](https://groups.google.com/u/1/g/cms-ab2d-api).

When emailing the team please include the following information concerning the context of issues:

- What operating system is in use by the machine (Windows or Linux/Mac)?
- What IP address does the machine have (see Verifying Setup)?
- What is the HTTP status code (403, 400, etc.) that is being received, if applicable?
- A description of the issue including the stage of the export causing the issue.
- Any additional logs (be extremely careful with the logs you provide). Refer to the Security Practices section as to which pieces of information are sensitive.

When emailing the team ***do not*** include any form of the credentials provided including any encoded form. Please double check any logs provided to make sure the logs do not contain the organization’s credentials.
