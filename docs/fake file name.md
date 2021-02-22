## Important note

Documentation/Content Organization will become wild if we do not maintain a process. Make suggestions to make the process better! Hooray for democracy. But please, respect the current process until we, as a team, decide on something else. 


## Customer Documentation Folder Structure


### GitHub

This is documentation that gets placed on GitHub (big surprise). But what that really means is: this is documentation that is provided to PDPs only via private access and only after they have received access to production (been given credentials). 


### Website

This is documentation that goes on the AB2D static site. At this point, this is only Sandbox documentation. It may grow. 


### Email Documents

These are email templates that we’ll be using to try to move the needle on some of our P1 OKRs.


### Subfolders

Within each folder (GitHub/Website) are X folders: 



1. Draft: _draft versions of document before they get reviewed by CMS_
2. Ready for CMS review: this document is ready for CMS eyes. _There are no comments, nothing in suggestion mode is left._
3. Approved: _this doc has the CMS seal of approval_
4. Currently published: _this is the version of doc currently found on GitHub_.
5. Supplanted by newer version: _this doc was on GitHub but is no longer. Move over._
6. Legacy: ignore. Older process, older docs.

**Important**: Our processes assume that a document does not physically move across folders getting renamed but rather a new version of that document gets created for each folder. The only exception is when a document is supplanted and it then physically moves from Currently Published to Supplanted by Newer Version.


## Airtable

Everyone on the AB2D team has access to the Document [Airtable](https://airtable.com/invite/l?inviteId=invmz46Y30h6gACCu&inviteToken=bdab9dfade65633621828a7f02e971549f671ac7e91b3544915a49212ffb4b62) located in the Customer Documentation Google Drive. Use the ab2d-service gmail account to access and the PW in the 1Password application.**<span style="text-decoration:underline;"> If you don’t have access do not go any further than this point until you do.</span>**


## How the process works: GitHub docs and Airtable

A document is either new or updated. 


### Creating a new document



1. Create a Jira ticket
2. Name the newly created document with the title/# of the Jira ticket + the date the document was created + a suffix of either “G” or “W” (for GitHub or Website) + “Draft”

Here is an example of how to name your document: `AB2D-3205_20200219_G_Draft`



3. This document goes to live in the Draft folder.
4. Update in Airtable:
    1. Doc name = name above (`AB2D-3205_20200219_G_Draft)`
    2. Description = working name of document, what it’s about
    3. Google Drive folder = GitHub
    4. Stage = Draft
    5. Date you are adding it to this Stage
    6. Sandbox or Production document
5. Make changes internally **IN SUGGESTION MODE**.
6. Once the team decides (however everyone wants to do this) that the document is ready to go to CMS for approval: 
    7. Place the clean version in the next folder: Ready for CMS Review
    8. **DO NOT** change the status in the Airtable. Add a new line with the following information: 
        1. Doc name = name above (`AB2D-3205_20200219_G_Ready)`
        2. Description = working name of document, what it’s about
        3. Google Drive folder = GitHub
        4. Stage = Ready for CMS Review
        5. Date you are adding it to this Stage

This way we can track the document across time.



    9. Download the document into Word. This is the version that goes to Andy/Yadira.
7. Send Word version to Andy/Yadira.
8. Once changes are received from Andy/Yadira: 
    10. Save their changes in Approved with “_Andy_edts” suffix like this: `AB2D-3205_20200219_G_Ready_Andy_edits`
    11. Place this document in Ready folder
    12. Wait for Yadira’s comments and then do the same:

        ```
        AB2D-3205_20200219_G_Ready_Yadira_edits
        ```


        6. So now in the Ready folder we should have 4 versions: Google doc Ready version, Word doc Ready version, and returned with CMS edits Ready version.
    13. Update the `AB2D-3205_20200219_G_Ready` with the` AB2D-3205_20200219_G_Ready_CMS_edits `document. This will give you: `AB2D-3205_20200219_G_Approved.`
9. Place the` AB2D-3205_20200219_G_Approved `document in the Approved folder and update the Airtable just as you did above.
10. Your next steps will be adding this document to GitHub: covered below.


### Updating an existing document

Because we are using Jira ticket #’s to track our documentation, we do not have to be concerned about duplicate document names when updating a document.

Two major things to keep in mind are:



1. Following GitHub processes correctly and
2. Remember that only 1 version of a document can live in the Currently Published folder at a time. This is different from all other folders. 

Example: 



*   If the FAQ doc `AB2D-1234_20221231_G_Approved` moves from the approved folder to the Currently published folder, a new version of that doc is created. This is not different than before. 
*   Then when we decide this is doc that gets published we make a copy, rename to  `AB2D-1234_20221231_G_CurrentlyPublished` and place in the Currently Published folder.
*   When a new document comes a long to supplant this one, that is the only time we physically move the document and rename it to  `AB2D-1234_20221231_G_Supplanted_dateofreplacement (ex: AB2D-1234_20221231_G_Supplanted_20230210)`


## How the process works: GitHub docs and using GitHub


### Creating a new document

A document has been approved by CMS PO and Director. It is ready for GitHub. 

Document has to be placed into GitHub via markdown. 

Install Docs to Markdown addon to GoogleDocs in order to do this easily. 

After doc is in Markdown move to 


### Updating an existing document


## How the process works: Website docs and Airtable


### Creating a new document


### Updating an existing document


## How the process works: Website docs and the static site
