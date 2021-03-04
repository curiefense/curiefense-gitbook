# URL Maps

![](../../.gitbook/assets/url-maps%20%281%29.png)

The input controls at the top of this page are described here: [Policies & Rules Entry Administration](./#entry-administration). Specific editing of a URL Map is described below.

## Overview

This page specifies a list of URLs and the security policies assigned to them.

Every incoming HTTP/S request targets a specific URL. Curiefense finds the best match for that URL in the URL Maps, and applies the security policies defined for it.

{% hint style="info" %}
The "best match" is determined by regex evaluation. \(The order in which the URLs are listed in the interface does not matter.\) If no matching definition is found, Curiefense applies the rulesets from the default definition.
{% endhint %}

## Components of a URL Map

A URL Map consists of:

* **Host definition**: The \(sub\)domain\(s\) within which the Path Maps will be found.
* **Path Maps**: one or more paths, and the security policies which will be applied to them.

## Creating a URL Map

To add a new URL Map, use the buttons at the top of the window to duplicate an existing one or create a new one. Then fill in these fields.

| Field | Value |
| :--- | :--- |
| **\(unlabeled\)** | The name of the URL Map is displayed for editing on the upper left. |
| **Matching Names** | A regex for the subdomain\(s\) and/or domain\(s\). |

## Editing its Path Maps

To add a new Path Map, select an existing one in the middle of the window, expand it, and select **Fork Profile** at the bottom. The existing one will be cloned, and the new one will be displayed for editing.

{% hint style="info" %}
Note that the buttons at the top of the window are for administering URL Maps \(which generally correspond to domains\). Administering Path Maps \(for paths and URLs within the specified domain\) is done in the middle of the window.
{% endhint %}

To edit an existing Path Map definition, click on the "expand" button at the end of its listing. The edit window shown below will appear.

![](../../.gitbook/assets/url-maps-editing%20%281%29.png)

| **Field** | **Value** |
| :--- | :--- |
| **Name** | A descriptive label for use within the interface. |
| **Match** | An expression for the path, expressed as PCRE \(Perl Compatible Regular Expressions\). |
| **WAF** | The [WAF Policy](waf-policies.md) applied to this path. Its name will be displayed in green if it is active; if displayed in red, it is currently disabled. |
| **ACL** | The [ACL Policy](acl-policies.md) applied to this path. Its name will be displayed in green if it is active; if displayed in red, it is currently disabled. |
| **RL** | The number of [Rate Limits](rate-limits.md) assigned to this resource. |

In addition to editing the fields discussed above, this window also provides the ability to:

* Deactivate the WAF Policy \(by unselecting its Active Mode checkbox\).
* Deactivate the ACL Policy \(by unselecting its Active Mode checkbox\).
* Assign \(or remove\) Rate Limit rules to this Path Map, via the **+** button \(or **remove** button\). The **+** button will only be shown if there are unassigned Rate Limit rules available.
* Create a copy of this Path Map, and open it for editing, via the **Fork Profile** button.

