# Claims Representation Details

AB2D and its upstream data source generate and add fields to claims data in an
effort to help make managing and providing those claims easier. Organizations using AB2D will have to utilize
these fields when:



*   [Mapping a claim to a beneficiary](#identifying-patients)
*   [Checking whether a claim is active or cancelled](#claim-status)
*   [Claim identifiers and identifying updates to claims](#identifying-claims-and-claim-versions)
*   [Identifying updated claims](#detecting-updates-to-existing-claims)
*   [Identifying cancelled claims](#resolving-cancelled-claims)
*   [Identifying duplicate claims](#detecting-duplicate-claims)

### Important AB2D Claims Fields


<table>
  <tr>
   <td>Field
   </td>
   <td>Description
   </td>
   <td>EOB Location
   </td>
   <td>For More Details
   </td>
  </tr>
  <tr>
   <td>Claim Group
   </td>
   <td>Unique identifier of a claim which is the same across claim updates.
<p>
This field can be used to group together a family of claims.
   </td>
   <td>eob.identifier[]
<p>
- found in list of identifiers
   </td>
   <td><a href="#identifying-claims-and-claim-versions">Identifying Claims and Claim Versions</a>
   </td>
  </tr>
  <tr>
   <td>Claim ID
   </td>
   <td>Unique identifier of a single version of a claim. Not the same across updates
   </td>
   <td>eob.identifier[]
<p>
- found in list of identifiers
   </td>
   <td><a href="#identifying-claims-and-claim-versions">Identifying Claims and Claim Versions</a>
   </td>
  </tr>
  <tr>
   <td>Claim Status
   </td>
   <td>If you are, however, viewing one version of a claim within the context of a claim family and the last version is canceled, then the claim family is canceled. 
   </td>
   <td>eob.status
<p>
- “active” or “cancelled”
   </td>
   <td><a href="#claim-status">Claim Status</a>
   </td>
  </tr>
  <tr>
   <td>Claim Last Updated
   </td>
   <td>The last time any modification (new version) of a claim was received
   </td>
   <td>eob.meta.lastUpdated
   </td>
   <td><a href="#last-updated">Last Updated</a>
   </td>
  </tr>
  <tr>
   <td>MBI
   </td>
   <td>Medicare Beneficiary Identifier
<p>
(Medicare standard identifier)
   </td>
   <td>eob.extension
   </td>
   <td><a href="#identifying-patients">Identifying Patients</a>
   </td>
  </tr>
  <tr>
   <td>Claim Version
   </td>
   <td>NOT a field received from the AB2D API but an important concept which relates to all fields above.
   </td>
   <td>n/a
   </td>
   <td><a href="#identifying-claims-and-claim-versions">Identifying Claims and Claim Versions</a>
   </td>
  </tr>
</table>



## The Business Concept of a Claim vs. a Claim Object

Each claim made by a provider translates into a claim object in AB2D’s upstream data source. **This claim object is immutable.
Any change to the claim leads to the creation of a completely new claim object.** 
This new claim object “version” along with prior versions are stored by AB2D’s upstream data source and provided by the AB2D API.

Thus, the AB2D API can/will provide multiple claim objects that map to a single claim made by a provider.

Importantly, the following is true about claim objects and AB2D:



*   AB2D reports all claim objects regardless of whether those claim objects are the latest version available.
*   Only the latest claim object will have Claim Status = ‘active’. All other claim objects provided will be marked Claim Status = “canceled”.
*   Updates can cause the creation of new claim objects at any time.
*   Organizations using AB2D are responsible for determining the most recent claim object.


## Identifying Claims and Claim Versions

There are two primary Claim Identifiers used to logically identify and map relationships between business claims and claim object versions:


<table>
  <tr>
   <td>Claim Identifier
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>Claim Group ID
   </td>
   <td>The unique identification number of the business concept of a claim made by a provider.  \
While each change to a claim will result in a new claim object, each of these “versions” falls under the same Claim Group ID. . The Claim Group ID is the parent or common identifier linking related claim object versions.
   </td>
  </tr>
  <tr>
   <td>Claim ID
   </td>
   <td>The unique identification number of a single claim object “version”.  \
Each change to a claim will result in a new claim object and result in the assignment of a new unique Claim ID. Each of
these related Claim ID will share a common Claim Group ID. A Claim ID may have multiple claim lines.
   </td>
  </tr>
</table>



### Example Claim Versions

A claim object enters the AB2D data source on January 1st, 2020. This is the first claim object “version” (Claim Version 1) of the claim to arrive.

The claim comes with the following identifiers:



1. Claim Group: 99995
2. Claim ID: 321

A month later on February 1st, 2020 the claim is updated and a new claim object version arrives.
The new claim version has an additional line.

This new claim version (Claim Version 2) comes with the following identifiers:



1. Claim Group: 99995
2. Claim ID: 789

Notice two things:



1. The claim group stays the same between versions of the claim.
2. The Claim ID changes between versions of the claim.

Note: Claim Version is not a field in the data and will not be reported to PDPs.


<table>
  <tr>
   <td><strong>Claim Group ID</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="7" >99995
   </td>
   <td rowspan="3" >01/01/2020
   </td>
   <td rowspan="3" >321
   </td>
   <td>ABCD
   </td>
  </tr>
  <tr>
   <td>DEFG
   </td>
  </tr>
  <tr>
   <td>HIJK
   </td>
  </tr>
  <tr>
   <td rowspan="4" >02/01/2020
   </td>
   <td rowspan="4" >789
   </td>
   <td>ABCD
   </td>
  </tr>
  <tr>
   <td>DEFG
   </td>
  </tr>
  <tr>
   <td>HIJK
   </td>
  </tr>
  <tr>
   <td>LMNO
   </td>
  </tr>
</table>



### URI vs URL

All URLs below are highlighted in blue. Else they are a URI (uniform resource identifier). See [URI vs URL](https://danielmiessler.com/study/difference-between-uri-url/) for more information.


### Claim Group

The claim group is always one of a list of IDs found in the “identifier” field. The Claim Group ID remains the same between all versions of a claim. It can be used to group together a “family” of claims.

Basic Facts:



1. Format: a positive number
2. Location: found in the list of identifiers (eob.identifier[])
3. Claim Extension System: https://bluebutton.cms.gov/resources/identifier/claim-group

For more information see:



1. Identifier explanation: [http://hl7.org/fhir/R4/explanationofbenefit-definitions.html#ExplanationOfBenefit.identifier](http://hl7.org/fhir/R4/explanationofbenefit-definitions.html#ExplanationOfBenefit.identifier)


#### Example JSON

```json5
{
    ...
    "identifier": [
        {
          "system": "https://bluebutton.cms.gov/resources/variables/clm_id",
          "value": "-10000521860"
        },
        {
          "system": "https://bluebutton.cms.gov/resources/identifier/claim-group",
          "value": "7653956538"
        }
    ],
    ...
}
```

### Claim ID

The claim id is found in every ExplanationOfBenefits as one of a list of IDs found in the “identifier” field ().

Basic Format:



1. Format: a positive number
2. Location: found in the list of identifiers (eob.identifier[])
3. Claim Extension System: [https://bluebutton.cms.gov/resources/variables/clm_id](https://bluebutton.cms.gov/resources/variables/clm_id)

For more information:



1. FHIR Identifier Explanation: [http://hl7.org/fhir/R4/explanationofbenefit-definitions.html#ExplanationOfBenefit.identifier](http://hl7.org/fhir/R4/explanationofbenefit-definitions.html#ExplanationOfBenefit.identifier)
2. Claim ID Explanation: [https://bluebutton.cms.gov/resources/variables/clm_id](https://bluebutton.cms.gov/resources/variables/clm_id)


#### Example

```json5
{
    ...
    "identifier": [
        {
          "system": "https://bluebutton.cms.gov/resources/variables/clm_id",
          "value": "-10000521860"
        },
        {
          "system": "https://bluebutton.cms.gov/resources/identifier/claim-group",
          "value": "7653956538"
        }
    ],
    ...
}
```

## Claim Status

**Claims reported by AB2D API have only two potential values for Claim Status**
Claims are either “active” or “cancelled”. **If a claim object is “active” then it is the latest version of the
claim received. If a claim is “cancelled”, then the claim object is not the latest version of the claim or the claim itself has been revoked.**

Basic Information on Status:



1. Format: string with one of two values “active” or “cancelled”
2. Location: found directly on the claim
3. URI: [http://hl7.org/fhir/explanationofbenefit-status](http://hl7.org/fhir/explanationofbenefit-status)

For more information:



1. FHIR Specification for Claim Status: [http://hl7.org/fhir/R4/valueset-explanationofbenefit-status.html](http://hl7.org/fhir/R4/valueset-explanationofbenefit-status.html).


## Last Updated

Changes to lastUpdated do not necessarily reflect changes to the claim itself. The lastUpdated field is a metadata field reflecting the last time a change was received to a particular claim.

Basic Information on Last Updated:



1. Format: ISO datetime with a timezone
2. Location: Found in claims metadata (eob.meta.lastUpdated)
3. URI: [http://hl7.org/fhir/R4/resource-definitions.html#Meta.lastUpdated](http://hl7.org/fhir/R4/resource-definitions.html#Meta.lastUpdated)

For more information:



1. Created: [http://hl7.org/fhir/R4/explanationofbenefit-definitions.html#ExplanationOfBenefit.created](http://hl7.org/fhir/R4/explanationofbenefit-definitions.html#ExplanationOfBenefit.created)
2. Common FHIR Metadata: [http://hl7.org/fhir/R4/resource-definitions.html#Resource.meta](http://hl7.org/fhir/R4/resource-definitions.html#Resource.meta)


## Identifying Patients

Each claim contains patient identifiers necessary to map a claim to a specific patient.
The primary identifier used to associate a claim with a patient is the Medicare Beneficiary Identifier (MBI).
**As part of each claim, AB2D reports all MBIs, known to the system, for a patient including the current MBI for a patient and historic MBIs used in the past for a patient.**


<table>
  <tr>
   <td>Identifier Name
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>Current MBI
   </td>
   <td>The current Medicare Beneficiary Identifier (MBI) for a patient. Uniquely identifies a patient across CMS.
   </td>
  </tr>
  <tr>
   <td>Historic MBI
   </td>
   <td>An inactive Medicare Beneficiary Identifier (MBI) for a patient. Uniquely identifies a patient across CMS.
   </td>
  </tr>
</table>



### MBI Extension Structure

Extensions referring to identifiers will have the following structure:



1. FHIR Identifier Extension URI: [http://hl7.org/fhir/StructureDefinition/elementdefinition-identifier](http://hl7.org/fhir/StructureDefinition/elementdefinition-identifier)
2. MBI Identifier System: http://hl7.org/fhir/sid/us-mbi
3. Identifier location: extension.valueIdentifier.value
4. Identifier Currency Code Extension URI:  https://bluebutton.cms.gov/resources/codesystem/identifier-currency


#### JSON Example

```json5
{
  // (1) url is always the same
  "url": "http://hl7.org/fhir/StructureDefinition/elementdefinition-identifier",
  "valueIdentifier": {

      // (4) whether the MBI is currently in use or a historical value will be found as an extension with the url
      "extension": [
          {
            "url": "https://bluebutton.cms.gov/resources/codesystem/identifier-currency",
            "valueCoding": {
              "code": "current"
            }
          }
      ],

      // (2) system will always be
      "system": “http://hl7.org/fhir/sid/us-mbi”
      // (3) the actual MBI value will always be found here
      "value": "7S94E00AA00"
  }
}
```


### Current MBI

The current Medicare Beneficiary Identifier (MBI) used to identify a patient.



1. Format: A string following the CMS standards ([https://www.cms.gov/Medicare/New-Medicare-Card/Understanding-the-MBI.pdf](https://www.cms.gov/Medicare/New-Medicare-Card/Understanding-the-MBI.pdf))
2. Location: found in the list of extensions provided with the EOB
3. Identifier System: http://hl7.org/fhir/sid/us-mbi


### Historic MBI

A Medicare Beneficiary Identifier (MBI), which in the past was used to identify the patient.



1. Format: A string following the CMS standards see [https://www.cms.gov/Medicare/New-Medicare-Card/Understanding-the-MBI.pdf](https://www.cms.gov/Medicare/New-Medicare-Card/Understanding-the-MBI.pdf)
2. Location: found in the list of extensions provided with the EOB
3. Identifier System: http://hl7.org/fhir/sid/us-mbi


### Determining Current vs. Historic

The list of extensions provided with each MBI includes an extension containing whether the MBI is current or historic.

If an MBI is current then the following example is representative.

```json5
{
    "extension": [
        {
          "url": "http://hl7.org/fhir/StructureDefinition/elementdefinition-identifier",
          "valueIdentifier": {
            "extension": [
              {
                "url": "https://bluebutton.cms.gov/resources/codesystem/identifier-currency",
                "valueCoding": {
                  "code": "current"
                }
              }
            ],
            "system": "http://hl7.org/fhir/sid/us-mbi",
            "value": "7S94E00AA00"
          }
        }
    ]
}
```

Whereas if the MBI was historic then the value would be switched to historic.

```json5
{
    "extension": [
        {
          "url": "http://hl7.org/fhir/StructureDefinition/elementdefinition-identifier",
          "valueIdentifier": {
            "extension": [
              {
                "url": "https://bluebutton.cms.gov/resources/codesystem/identifier-currency",
                "valueCoding": {
                  "code": "historic"
                }
              }
            ],
            "system": "http://hl7.org/fhir/sid/us-mbi",
            "value": "7S94E00AA00"
          }
        }
    ]
}
```

## Detecting Updates to Existing Claims

AB2D reports the most recent data available to it. Updates to existing claims may arrive at any time.

**To detect updated claims look for a claim group with multiple Claim IDs. Only one claim version (one Claim ID)
will be marked active and that is the latest version.
If all claim versions are marked as canceled then the entire claim family group is not active.**

To detect updated claims the following fields on each claim must be tracked:



1. Claim Group ID: Unique ID of the claim family which is the same regardless of the claim version
2. Claim ID: Unique ID of the claim object which changes for each version of a claim
3. Updated Date: When the most recent change was made to the claim
4. Status: The current status of the claim (active or canceled)


### Example Scenario

In the example, a single claim will be tracked through several evolutions:

Claim Update


<table>
  <tr>
   <td>Claim Update
   </td>
   <td>Details
   </td>
  </tr>
  <tr>
   <td>Update 1
   </td>
   <td>Claim 99995 is received by AB2D on January 1st, 2020
   </td>
  </tr>
  <tr>
   <td>Update 2
   </td>
   <td>An update to Claim 99995--adding another line--is received by AB2D on February 10th, 2020
   </td>
  </tr>
  <tr>
   <td>Update 3
   </td>
   <td>An update to Claim 99995--removing the line--is received by AB2D on March 20th, 2020
   </td>
  </tr>
</table>


In the example, four exports are run using the _since parameter to limit duplicate data. The four exports are described below.


<table>
  <tr>
   <td>Export
   </td>
   <td>Run On
   </td>
   <td>Since
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>Export 1
   </td>
   <td>January 31st, 2020
   </td>
   <td>December 31st, 2019
   </td>
   <td>Provide all claims information and updates received in January
   </td>
  </tr>
  <tr>
   <td>Export 2
   </td>
   <td>February 28th, 2020
   </td>
   <td>January 31st, 2020
   </td>
   <td>Provide all claims information and updates received in February
   </td>
  </tr>
  <tr>
   <td>Export 3a
   </td>
   <td>March15th, 2020
   </td>
   <td>February 14th, 2020
   </td>
   <td>Provide all claims information and updates received after the first half of February
   </td>
  </tr>
  <tr>
   <td>Export 3b
   </td>
   <td>March31st, 2020
   </td>
   <td>February 28th, 2020
   </td>
   <td>Provide all claims information and updates received in March
   </td>
  </tr>
</table>


The dates used in this example are just random dates. Organizations cannot pull data before the date they legally attested for the data and cannot pull data before January 1st, 2020.


### Updated Claims Example- Claim Scenario-Export Details


#### Export 1 12/31/2019 - 01/31/2020

On January 31st, 2020 XYZ runs its first export. The export uses a _since date of December 31st, 2019. The _since date tells AB2D to report all claims information received between December 31st, 2019 and January 31st, 2020.

The export pulls a claim version with Claim Group ID 99995. At this time, the only version of the claim available to AB2D is 12987987. This version (corresponding to claim _Update #1_) is the latest claim version available and is marked active.


<table>
  <tr>
   <td><strong>Claim Group</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
   <td><strong>Claim Query Code</strong>
   </td>
   <td><strong>Received</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>status as of 01/31/2020</strong>
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
</table>



#### Export 2 01/31/2020 - 02/28/2020

On February 28th, 2020 XYZ runs a second export. The export uses a _since date of January 31st, 2020. The _since date tells AB2D to report all claims information received between January 31st, 2020 and February 28th, 2020.

The export pulls two claim versions with Claim GroupID 99995 including the old, cancelled claim version and the newest, active claim version correspond to claim _Update #2_.

Claim version 12987987 is pulled for a second time. Here is why:



1. The claim status was changed to “cancelled” on February 10th, 2020
2. Changing the status triggered a change in the Last Updated field to February 10th, 2020
3. The _since date--January 31st, 2020--is before the Last Updated date so the claim version is pulled

Claim version 54689123 is pulled for the first time. Here is why:



1. The claim version was received on February 10th, 2020
2. Receiving the claim sets the Last Updated date to February 10th, 2020
3. The _since date--January 31st, 2020--is before the Last Updated date so the claim version is pulled.

<table>
  <tr>
   <td>
<strong>Claim Group</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
   <td><strong>Claim Query Code</strong>
   </td>
   <td><strong>Received</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>status as of 02/28/2020</strong>
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>LMNO
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>active
   </td>
  </tr>
</table>



#### Export 3a 02/28/2020 - 03/15/2020

On March 15th, 2020 XYZ runs a second export. The export uses a _since date of February 28th, 2020. The _since date tells AB2D to report all claims information received between February 28th, 2020 and March 15th, 2020.

The export pulls no information on claim 99995. AB2D has received no updates for the time interval and so no record are pulled.


#### Export 3b 02/28/2020 - 03/31/2020

On March 31st, 2020 XYZ runs a second export. The export uses a _since date of February 28th, 2020. The _since date tells AB2D to report all claims information received between February 28th, 2020 and March 31st, 2020.

The export pulls two claim versions with Claim Group ID 99995 including the second, cancelled claim version and the third, active claim version. These claim versions reflect to claim _Update #3_.

Claim version 54689123 is pulled for a second time. Here is why:



1. The claim status was changed to “cancelled” on March 20th, 2020
2. Changing the status triggered a change in the Last Updated field to March 20th, 2020
3. The _since date--February 20th, 2020--is before the Last Updated date so the claim version is pulled

Claim version 54689123 is pulled for the first time. Here is why:



1. The claim version was received on March 20th, 2020
2. Receiving the claim sets the Last Updated date to March 20th, 2020
3. The _since date--February 28th, 2020--is before the Last Updated date so the claim version is pulled.

<table>
  <tr>
   <td>
<strong>Claim Group</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
   <td><strong>Claim Query Code</strong>
   </td>
   <td><strong>Received</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>status as of 03/31/2020</strong>
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>03/31/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>03/31/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>03/31/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>LMNO
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>03/31/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>34543
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>03/31/2020
   </td>
   <td>03/31/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>34543
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>03/31/2020
   </td>
   <td>03/31/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>34543
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>03/31/2020
   </td>
   <td>03/31/2020
   </td>
   <td>active
   </td>
  </tr>
</table>



## Resolving Cancelled Claims

AB2D provides claims based on the current information available. A claim may later be cancelled at any time.

**A claim is cancelled when all versions of the claim are marked as cancelled.**

To detect updated claims the following fields on each claim must be tracked:



1. Claim Group ID: Unique ID of the claim which is the same regardless of the claim version
2. Claim ID: Unique ID of the claim version which changes for each version of a claim
3. Updated Date: When the most recent change was made to the claim
4. Status: The current status of the claim


### Example Scenario

In the example, a single claim will be tracked through several evolutions:


<table>
  <tr>
   <td>Claim Update
   </td>
   <td>Details
   </td>
  </tr>
  <tr>
   <td>Update 1
   </td>
   <td>Claim 99995 is received by AB2D on January 1st, 2020
   </td>
  </tr>
  <tr>
   <td>Update 2
   </td>
   <td>An update to Claim 99995 is received by AB2D on February 28th, 2020
   </td>
  </tr>
</table>


In the example, two exports are run using the _since parameter to limit duplicate data. The two exports are described below.


<table>
  <tr>
   <td>Export
   </td>
   <td>Run On
   </td>
   <td>Since
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>Export 1
   </td>
   <td>January 31st, 2020
   </td>
   <td>December 31st, 2019
   </td>
   <td>Provide all claims information and updates received in January
   </td>
  </tr>
  <tr>
   <td>Export 2
   </td>
   <td>February 28th, 2020
   </td>
   <td>January 31st, 2020
   </td>
   <td>Provide all claims information and updates received in February
   </td>
  </tr>
</table>


The dates used in this example are just random dates. Organizations cannot pull data before the date they legally attested for the data and cannot pull data before January 1st, 2020.

Cancelled Claims Example- Claim Scenario-Export Details Organization XYZ decides to start exporting claims at the beginning of 2020.


#### Export 1 12/31/2019 - 01/31/2020

On January 31st, 2020 XYZ runs its first export. The export uses a _since date of December 31st, 2019. The _since date tells AB2D to report all claims information received between December 31st, 2019 and January 31st, 2020.

The export pulls a claim version with Claim Group ID 99995. At this time the only version of the claim available to AB2D is 12987987. This version (corresponding to claim _Update #1_) is the latest claim version available and is marked active.


<table>
  <tr>
   <td><strong>Claim Group</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
   <td><strong>Claim Query Code</strong>
   </td>
   <td><strong>Received</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>status as of 01/31/2020</strong>
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
</table>



#### Export 2 01/31/2020 - 02/28/2020

On February 28th, 2020 XYZ runs a second export. The export uses a _since date of January 31st, 2020. The _since date tells AB2D to report all claims information received between January 31st, 2020 and February 28th, 2020.

The export pulls two claim versions with Claim Group ID 99995 including the old, cancelled claim version and the newest, also cancelled claim version correspond to claim _Update #2_. Thus, all versions of the claim are cancelled

Claim version 12987987 is pulled for a second time. Here is why:



1. The claim status was changed to “cancelled” on February 10th, 2020
2. Changing the status triggered a change in the Last Updated field to February 10th, 2020
3. The _since date--January 31st, 2020--is before the Last Updated date so the claim version is pulled

Claim version 54689123 is pulled for the first time. Here is why:



1. The claim version was received and marked as cancelled on February 10th, 2020
2. Receiving the claim sets the Last Updated date to February 10th, 2020
3. The _since date--January 31st, 2020--is before the Last Updated date so the claim version is pulled.

<table>
  <tr>
   <td>
<strong>Claim Group</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
   <td><strong>Claim Query Code</strong>
   </td>
   <td><strong>Received</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>status as of 02/28/2020</strong>
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>54689123
   </td>
   <td>LMNO
   </td>
   <td>final
   </td>
   <td>02/10/2020
   </td>
   <td>02/10/2020
   </td>
   <td>cancelled
   </td>
  </tr>
</table>



## Detecting Duplicate Claims

Duplicate claims occur when an organization pulls the same claim version twice and all field values are the same. Even using the _since parameter, the AB2D API makes no guarantee that this will never happen.

To detect duplicate claims the following fields on each claim must be tracked:



1. Claim Group ID: Unique ID of the claim family which is the same regardless of the claim version
2. Claim ID: Unique ID of the claim version which changes for each version of a claim


### Example Scenario

In the example, a single claim will be tracked:



1. Claim 99995 is received by AB2D on January 1st, 2020

In the example, two exports are run using the _since parameter. These exports are for overlapping time periods so duplicate information will be received. The two exports are described below.


<table>
  <tr>
   <td>Export
   </td>
   <td>Run On
   </td>
   <td>Since
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>Export 1
   </td>
   <td>January 31st, 2020
   </td>
   <td>December 31st, 2019
   </td>
   <td>Provide all claims information and updates received in January
   </td>
  </tr>
  <tr>
   <td>Export 2
   </td>
   <td>February 28th, 2020
   </td>
   <td>December 31st, 2019
   </td>
   <td>Provide all claims information and updates received in January and February
   </td>
  </tr>
</table>


The dates used in this example are just random dates. Organizations cannot pull data before the date they legally attested for the data and cannot pull data before January 1st, 2020.

Detecting Duplicate Claims Example- Claim Scenario-Export Details Organization XYZ decides to start exporting claims at the beginning of 2020.


#### Export 1 12/31/2019 - 01/31/2020

On January 31st, 2020 XYZ runs its first export. The export uses a _since date of December 31st, 2019. The _since date tells AB2D to report all claims information received between December 31st, 2019 and January 31st, 2020.

The export pulls a claim version with Claim Group ID 99995. At this time the only version of the claim available to AB2D is 12987987. This version is the latest claim version available and is marked active.


<table>
  <tr>
   <td><strong>Claim Group</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
   <td><strong>Claim Query Code</strong>
   </td>
   <td><strong>Received</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>status as of 01/31/2020</strong>
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
</table>



#### Export 2 01/31/2020 - 02/28/2020

On February 28th, 2020 XYZ runs a second export. The export uses a _since date of January 31st, 2020. The _since date tells AB2D to report all claims information received between January 31st, 2020 and February 28th, 2020.

The export pulls a claim version with Claim Group ID 99995. At this time the only version of the claim available to AB2D is 12987987. This version is the latest claim version available and is marked active.

Claim version 12987987 is pulled for a second time because the _since date remains the same.


<table>
  <tr>
   <td><strong>Claim Group</strong>
   </td>
   <td><strong>Claim ID</strong>
   </td>
   <td><strong>Claim Line ID</strong>
   </td>
   <td><strong>Claim Query Code</strong>
   </td>
   <td><strong>Received</strong>
   </td>
   <td><strong>Last Updated</strong>
   </td>
   <td><strong>status as of 02/28/2020</strong>
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>ABCD
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>DEFG
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
  <tr>
   <td>99995
   </td>
   <td>12987987
   </td>
   <td>HIJK
   </td>
   <td>final
   </td>
   <td>01/01/2020
   </td>
   <td>01/01/2020
   </td>
   <td>active
   </td>
  </tr>
</table>

