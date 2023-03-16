

# Welcome to the AB2D Document Repository

In this repository you will find documentation related to accessing production and retrieving production real
claims data using the AB2D (Claims Data to Part D Sponsors) API.

Please read the documentation thoroughly before connecting to the API as it outlines important prerequisites,
provides helpful hints and details step by step instructions for users to connect to the API.

The repository also provides sample scripts, which demonstrate how to pull data from the AB2D API. These clients are
provided as examples, but are fully functioning (with some modifications) in the production environment. When used in
production, the clients have the ability to download PII/PHI information. You should therefore ensure the environment
in which these scripts are run is secured in a way to allow for storage of PII/PHI. Additionally, when used in the
production environment the scripts will require use of your production credentials. As such, please ensure that your
credentials are handled in a secure manner and not printed to logs or the terminal. Ensuring the privacy of data is
the responsibility of each user and/or organization.


## 
**AB2D Documentation Repo Table of Contents**

[AB2D Documentation Repo](https://github.com/CMSgov/ab2d-pdp-documentation)



* [Accessing Production](./docs/Production%20Access.md)
  * Accessing the AB2D API for the first time
* [Claims Representation Details](./docs/Claims%20Representation%20Details.md)
  * How claims are represented in the AB2D API
* [Long Term API Usage Model](./docs/Long%20Term%20API%20Usage%20Model.md)
  * Using the AB2D API beyond your first job
* [Production User FAQs](./docs/Production%20User%20FAQs.md)
  * Often asked questions about the AB2D API

## 
**Sample Client Repos**


The sample client repos found below are direct links to the sample scripts referenced throughout our documentation:

*   [Python Sample Client](https://github.com/CMSgov/ab2d-sample-client-python/)
*   [Bash Sample Client](https://github.com/CMSgov/ab2d-sample-client-bash/)
*   [Powershell Sample Client](https://github.com/CMSgov/ab2d-sample-client-powershell/)

It is important to note that the AB2D team does not regularly maintain the sample clients. Additionally, a
best-effort was made to ensure the clients are secure but they have not undergone comprehensive formal security testing.
Each user/organization is responsible for conducting their own review and testing prior to implementation.


## 
**Asking for Help**

If you continue to have issues accessing production or have follow up questions not answered in this guide, please
email the AB2D team at [ab2d@cms.hhs.gov](mailto:ab2d@cms.hhs.gov) or join our Google Groups at:
[https://groups.google.com/u/1/g/cms-ab2d-api](https://groups.google.com/u/1/g/cms-ab2d-api).

When corresponding with the team please include the following information concerning the context of your issue:



*   What operating system is in use by the machine (Windows or Linux/Mac)?
*   What IP address does the machine have (see Verifying Setup)?
*   If applicable, what HTTP status code (403, 400, etc.) is being received?
*   A description of the issue including the stage of the export causing the issue.
*   Any logs that may help us in resolving the issue. Use caution when sharing any log files as they may contain sensitive information. Reference the Security Practices above regarding sensitive information.

Do not include your organization's production credentials in any form when working with the AB2D team. Please review all encoded content and/or logs before sharing with the team to ensure they do not contain your credentials.
