# Policies & Rules

Curiefense maintains its security parameters in Documents, which contain Entries. \(Read more about [Curiefense's data structures](../../#data-structures).\)

The Policies & Rules section is where Documents and their Entries are created, defined, and administered.

## Documents and traffic flow

Curiefense processes incoming requests according to this traffic flow:

* Each incoming request is inspected, and tags are assigned to it. For example, if the request's IP was found on the Spamhaus DROP list, it could be assigned a "spamhaus" tag. Some tags are generated automatically, while others are user-defined. \([Read more about tags](../../reference/tags.md).\)
* Next, Curiefense determines the security ruleset\(s\) that have been assigned to the request's target URL, and which match the tags. For example, there might be a ruleset defined for the "spamhaus" tag, or for the "devops" tag.
* Curiefense then enforces the ruleset\(s\), and takes the defined Action\(s\). For example, "block requests from Spamhaus-listed IPs", or "bypass devops requests from further filtering."

This process is based on the Documents as follows:

* \*\*\*\*[**Tag Rules** ](tag-rules.md)is the Document which defines tags for external lists and custom lists.
* \*\*\*\*[**ACL Policies**](acl-policies.md), [**Rate Limits**](rate-limits.md), and [**WAF Policies**](waf-policies.md) define security rulesets, i.e. the actions to take when specific tags and/or other criteria are observed.
* \*\*\*\*[**URL Maps**](url-maps.md) assign the security rulesets to internal URLs.

## Policies & Rules interface

![](../../.gitbook/assets/document-editor-acl-profiles%20%281%29.png)

This page is divided into three vertical sections. From top to bottom, they are:

* Administration
* Entry editing
* Versioning

Each is discussed below.

{% hint style="info" %}
After editing anything on this page, you must save your changes \(with the Save button on the upper right\) and then [publish them](../publish-changes.md).
{% endhint %}

## Entry Administration

The top section contains a toolbar with input controls. 

* **Configuration** pulldown: Selects the branch/configuration for editing.
* **Document** pulldown: Selects the Document to display for editing.
* **Entry** pulldown: Selects the Entry \(the ruleset\) that is being displayed for editing.
* **Duplicate** button: Makes a copy of the currently displayed Entry.
* **Download** button: Downloads the currently displayed parameters.
* **Add** button: Creates a new Entry of the currently selected type.
* **Save** button: Saves all changes that were made since the last Save action.
* **Delete** button. Deletes the currently selected Entry.

## Entry Editing

The UI in this section will vary, depending on the type of Document being edited. Each is discussed in more depth here:

* [ACL Policies](acl-policies.md)
* [Tag Rules](tag-rules.md)
* [Rate Limits](rate-limits.md)
* [URL Maps](url-maps.md)
* [WAF Policies](waf-policies.md)
* [WAF Rules](waf-rules.md)

At the bottom of this section, the URL of the current entry is shown in gray. It reflects the structure of the data. 

For example, `/conf/api/v1/configs/devops/d/urlmaps/e/__default__` shows that:

* The current Configuration \(or branch\) is devops.
* The current Document is urlmaps.
* The current Entry is \_\_default\_\_.

## **Document Versioning**

The bottom section of this page shows a history of versions for this Document. In a later version of Curiefense, this section will allow for reversions. In the current version, reversions are only available in the API and in the [Publish Configuration](../publish-changes.md) section of the UI.





