---
title: ElementFile element (ElementManifestReferences complexType)
manager: soliver
ms.date: 9/16/2015
ms.audience: Developer
ms.topic: reference
ms.prod: sharepoint
ms.localizationpriority: medium
ms.assetid: c25a5ef2-b9ef-926d-cf7b-9a91168c9f1f
---

# ElementFile element (ElementManifestReferences complexType) 

(AppHostWebFeatures)

> [!NOTE] 
> The string `app` appears as part of or all of some element, attribute, and file names because SharePoint Add-ins were originally called "apps for SharePoint." To ensure backward compatibility, the schemas have not been changed. 

## Element information

|   |   |
|---|---|
| **Element type**  | ElementManifestReference |
| **Namespace**  | `http://schemas.microsoft.com/sharepoint/` |
| **Schema file**  | apphostwebfeatures.xsd |

## Definition

```XML
    <xs:element name="ElementFile" type="ElementManifestReference" minOccurs="0" maxOccurs="unbounded"></xs:element>
```

## Elements and attributes

If the schema defines specific requirements, such as **sequence**, **minOccurs**, **maxOccurs**, and **choice**, see the definition section.

### Parent elements

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"><p>Element</p></th>
<th align="left"><p>Type</p></th>
<th align="left"><p>Description</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p><a href="applyelementmanifests-element-upgradeactionsgroup-groupapphostwebfeatures.md">ApplyElementManifests</a></p></td>
<td align="left"><p><a href="elementmanifestreferences-complextype-apphostwebfeatures.md">ElementManifestReferences</a></p></td>
<td align="left"><p></p></td>
</tr>
<tr class="even">
<td align="left"><p><a href="elementmanifests-element-featuredefinition-complextypeapphostwebfeatures.md">ElementManifests</a></p></td>
<td align="left"><p><a href="elementmanifestreferences-complextype-apphostwebfeatures.md">ElementManifestReferences</a></p></td>
<td align="left"><p></p></td>
</tr>
</tbody>
</table>

<br/> 

### Child elements

None.

<br/> 

### Attributes

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"><p>Attribute</p></th>
<th align="left"><p>Type</p></th>
<th align="left"><p>Required</p></th>
<th align="left"><p>Description</p></th>
<th align="left"><p>Possible values</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p>Location</p></td>
<td align="left"><p>RelativeFilePath</p></td>
<td align="left"><p>required</p></td>
<td align="left"><p></p></td>
<td align="left"><p>Values of the RelativeFilePath type.</p></td>
</tr>
</tbody>
</table>
<br/> 
<br/> 







