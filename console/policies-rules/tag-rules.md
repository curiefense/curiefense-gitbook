# Tag Rules

![](../../.gitbook/assets/tag-rules-google-crawlers.png)

The input controls at the top of this page are described here: [Policies & Rules](./#document-editor-interface)[ interface](./#document-editor-interface). Specific editing of a Tag Rules List is described below.

## Overview

Early in the [traffic evaluation process](../../reference/multi-stage-traffic-filtering.md), Curiefense assigns **Tags** to an incoming request. Subsequently, the Tags can be used to make decisions about how the request is processed. After processing, a request's Tags remain associated with it, and they are available for display in the [Access Log](../../analytics/access-log.md).

This page allows you to administer **Tag Rule Lists**, which are combinations of [user-defined Tags](../../reference/tags.md#user-defined-tags) and the criteria for assigning them to requests. 

{% hint style="info" %}
The actions that can be performed as a result of the Tag assignments are administered separately in [ACL Policies](acl-policies.md) and [Rate Limits](rate-limits.md).
{% endhint %}

Each Tag Rule List consists of:

* **Match conditions**: Definitions of possible characteristics that a request can match \(e.g., a list of IP addresses that it might originate from\), plus one or more logical operators to use when evaluating the match.
* **One or more Tags to assign** when a match occurs. 

For each incoming request, Curiefense will evaluate all active Tag Rule Lists. A single request will receive Tags from all Lists which match it.

## Metadata

| Field | Meaning |
| :--- | :--- |
| **Name** | A description that will be displayed within the Curiefense interface. |
| **Sections control** | The number of match condition sections, and the logical operator\(s\) to use if there is more than one. See discussions below about Single-Section Lists and Multiple-Section Lists. |
| **Tags** | One or more Tags \(separated by spaces\) that will be assigned to requests if the match conditions are fulfilled. Example: `internal team-devops`. |
| **Source** | The source of the match conditions. See discussion below. |
| **Active** | By default, the Tag Rule List will be applied to incoming requests. To prevent this, unselect the checkbox. |
| **Notes** | An optional field for including additional information. |

### Source

Tag Rule Lists can be either Internet-sourced or self-managed.

* **Internet-sourced** Lists are based upon online sources \(e.g., Spamhaus DROP lists\). To use an Internet-sourced list, enter its URL into the **Source** field. For example, to create a list based on the Spamhaus ASN DROP list, you would enter [https://www.spamhaus.org/drop/asndrop.txt](https://www.spamhaus.org/drop/asndrop.txt) and select **update now**. Curiefense will then populate the list automatically. 
* **Self-managed** Lists are created manually. The Source field entry will be `self-managed`.

Note that Internet-sourced lists are not editable within the interface, because they are obtained automatically; Curiefense updates them every 24 hours \(or you can force an immediate update by selecting **update now**\). Typically, these will be Single-Section Lists.

## Specifying Match Conditions

Match conditions are lists of criteria and the logical operators \(AND or OR\) to use when evaluating them. 

Match conditions are combined into sections. An additional logical operator defines the relationship between each section. 

## Single-Section Lists

Many Tag Rules Lists will have only section. Here's an example. 

![](../../.gitbook/assets/tag-rules-self-managed.png)

In the left-hand metadata, the **Sections control** shows that \(the number of\) **Sections** is 1. There is also a **Sections relation** toggle \(set to `AND` in the example above\). For a single-section list, this toggle is ignored.

On the right, at the top of the entries list, there is an **Entries relation** toggle \(set to `OR` in the example above\). This defines the logical operator to use when evaluating the criteria within the list. 

* `OR` means the **Tags** will be attached to each request that matches any of the entries in the list. 
* `AND` means the **Tags** will only be attached to requests that match all of the entries in the list.

### Adding a Match Condition

To add a match condition, select **Add entry** at the bottom of the list. The following dialog will appear.

![](../../.gitbook/assets/tag-rules-criteria-add-path.png)

For most of the categories, the dialog will appear as it is above. Multiple entries can be made at once, with each entry on a separate line. Each line contains the value, plus a pound sign \(\#\) followed by an **annotation** \(a label for display within the Curiefense interface\). Example:

![](../../.gitbook/assets/tag-rules-criteria-add-ip-examples.png)

For some categories, one entry can be made at a time, with each entry requiring multiple lines. Annotations are not preceded by a pound sign. 

![](../../.gitbook/assets/tag-rules-criteria-add-cookies.png)

{% hint style="info" %}
Match criteria are case-insensitive.
{% endhint %}

Here are some sample entries for the various categories. Notice that boolean operators are available.

![](../../.gitbook/assets/session-profiles-criteria-list.png)

Once created, these entries cannot be edited. If one needs to be modified, remove it and re-create it.

## Multiple-Section Lists

Match conditions can be combined into multiple sections. To add a section to an existing Tag Rules List, select **Create new section**.

Here's an example:

![](../../.gitbook/assets/tag-rules-two-sections.png)

Each additional section works the same as the initial section. Note that each section has its own **Entries relation** operator. 

However, a multiple-section list also includes a logical operator for the relationship\(s\) between the individual sections. In the example above, this is `AND`. Therefore, a request will receive the tag of `internal` if it matches any of the IPs in the first section, `AND` it also contains one of the HTTP methods specified in the second section. 

Note that the inter-section operator is shown in between the sections. To change the operator, select the toggle in the **Sections control** area on the left. 

