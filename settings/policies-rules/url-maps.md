# URL Maps

![](../../.gitbook/assets/url-maps-initial.png)

The input controls at the top of this page are described here: [Policies & Rules Entry Administration](./#entry-administration). Specific editing of a URL Map is described below.

## Overview

This page specifies a list of URLs and the security policies assigned to them.

Every incoming HTTP/S request targets a specific URL. Curiefense finds the best match for that URL in the URL Maps, and applies the security policies defined for it.

{% hint style="info" %}
The "best match" is determined by regex evaluation. The order in which the URLs are listed in the interface does not matter.
{% endhint %}

## Components of a URL Map

A URL Map consists of:

* **Host definition**: The \(sub\)domain\(s\) within which the Path Maps will be found.
* **Path Maps**: one or more paths, and the security policies which will be applied to them.

## The Default URL Map

![](../../.gitbook/assets/url-maps-default%20%281%29.png)

Every Curiefense deployment includes a default URL Map. If a request does not match any other URL Map, the default one is applied.

To ensure that a default always exists, the **Matching Name** and **Path Map** for this URL Map are not editable.

## Creating a URL Map

To add a new URL Map, use the buttons at the top of the window to duplicate an existing one or create a new one. Then fill in these fields.

| Field | Value |
| :--- | :--- |
| **Name** | The name of the URL Map for internal use. |
| **Matching Names** | A regex for the subdomain\(s\) and/or domain\(s\). |

When you create or revise a URL Map, it must have a unique combination of **Matching Name** and **Path Maps**. For this reason, when a new URL Map is created, the UI generates a unique Matching Name. 

![](../../.gitbook/assets/url-map-unique-matching-name.png)

This should be changed to a correct value before the URL Map is saved.

## Editing its Path Maps

A new URL Map will include a default Path Map. Clicking on it, or on the **expand** button at the end of its listing, will expand it for editing.

![](../../.gitbook/assets/url-maps-api-expanded%20%281%29.png)

To add a new Path Map, select an existing one, expand it, and select **Fork Profile** at the bottom. The existing one will be cloned, and the new one will be displayed for editing.

{% hint style="info" %}
Note that the buttons at the top of the window are for administering URL Maps \(which generally correspond to domains\). Administering Path Maps \(for paths and URLs _within_ the specified domain\) is done in the middle of the window.
{% endhint %}

### Path Map Fields

| **Field** | **Value** |
| :--- | :--- |
| **Name** | A descriptive label for use within the interface. |
| **Match** | An expression for the path, expressed as PCRE \(Perl Compatible Regular Expressions\).  |
| **WAF** | The [WAF Policy](waf-policies.md) applied to this path. Its name will be displayed in green if it is active; if displayed in red, it is currently disabled. |
| **ACL** | The [ACL Policy](acl-policies.md) applied to this path. Its name will be displayed in green if it is active; if displayed in red, it is currently disabled. |
| **RL** | The number of [Rate Limits](rate-limits.md) assigned to this resource. |

In addition to editing the fields discussed above, this window also provides the ability to:

* Activate or deactivate the WAF Policy \(by toggling its Active Mode checkbox\).
* Activate or deactivate the ACL Policy \(by toggling its Active Mode checkbox\).
* Assign an existing Rate Limit rule to this Path Map, via the **+** button or selecting the link \("_To attach an existing rule, click here_."\), then selecting **add**. \(The **+** button will only be shown if there are unassigned Rate Limit rules available.\)
* Create a new Rate Limit rule for this Path Map, by selecting the link \("_To create a new rate-limit rule, click here_."\)
* Remove an assigned Rate Limit Rule by selecting **remove**.
* Create a copy of this Path Map, and open it for editing, via the **Fork Profile** button.

