# WAF Policies

![](../../.gitbook/assets/WAF-Policies.png)

The input controls at the top of this page are described here: [Policies & Rules Entry Administration](./#entry-administration). Specific editing of a WAF Profile is described below.

## Overview

A WAF Profile is a set of security policies that are used by the Curiefense WAF (Web Application Firewall). Every deployment includes a default WAF Profile, and additional Profiles can be created. 

Every URL that Curiefense protects has a WAF Profile assigned to it in [Security Policies](security-policies.md). (If none is assigned explicitly, the default is used.) A request sent to a URL might, or might not, be filtered according to the assigned Profile. 

#### Reasons why WAF filtering might not occur:

* The request was blocked before WAF filtering would have occurred. (Before the WAF is used, [several other stages of filtering occur first](../../reference/multi-stage-traffic-filtering.md).)
* The applicable [ACL Profile](acl-profiles.md) resulted in an Action of Bypass, which exempts the request from WAF filtering. 
* The WAF is not in [Active Mode](security-policies.md#editing-its-path-maps) for this URL.

## Input Characteristics

At the top of the page, the following values are defined for incoming requests.

| Constraint                    | Meaning                                                                                                                                                                                                                                                                |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Max Length**                | The maximum allowable length of a header, cookie, or argument.                                                                                                                                                                                                         |
| **Max Count**                 | The maximum number of headers, cookies, or arguments allowed.                                                                                                                                                                                                          |
| **Ignore Alphanumeric input** | When this is selected, the WAF will not inspect requests that only contain alphanumeric characters. This reduces computational overhead by not evaluating benign requests. (Hostile requests such as SQLi, XSS, etc., will contain some non-alphanumeric characters.)  |

## Content Filtering and Whitelisting

By default, an incoming request will be compared to all the [WAF Rules](waf-rules.md). If any parameter (any header, cookie, or argument within it) fails this evaluation, the request will be blocked.

However, parameters can be whitelisted and exempted from this filtering. For each parameter, this can be done in two ways:

* Full exemption is available by specifying a regex pattern which, if it matches the parameter's value, will exempt that parameter from WAF Rule evaluation.
* Partial exemption is available by specifying a list of WAF Rules which will not be evaluated, even if the regex pattern is not matched. 

Along with this content whitelisting, a "positive security" form of content filtering is also available. Curiefense can be configured to require certain content in a specified parameter, and reject requests that do not contain it.

## Parameter Content Constraints

The bottom part of the UI defines Curiefense's behavior for both whitelisting and content filtering for each parameter. 

In the following discussion, a **constraint** refers to the values in the UI input controls (_Parameter_, _Matching Value_, _Restrict?_, and _Exclude WAF Rule_) that have been specified for one parameter.

Each incoming request is processed like this:

![](<../../.gitbook/assets/WAF-Profile-flowchart (1).png>)

This behavior is defined in the following fields in the UI. 

| Field                | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Parameter**        | The parameter whose value will be compared to the **Matching Value**. This can be provided as a specific **Name** (e.g., _sessionid_), or as a **Regex** to match multiple parameters (e.g., _user\_.+_). Note that a Name will be marked with a capital "A", while a Regex will be marked with "\</>".                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Matching Value**   | A regex pattern. If a parameter's value matches it, the parameter will be exempted from WAF Rule filtering. If it does not match, the **Restrict?** option becomes relevant.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Restrict?**        | If a parameter does not match the Matching Value, and **Restrict?** is selected, then the request is blocked. If **Restrict?** is not selected, then the **Exclude WAF Rule** option becomes relevant.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Mask?**            | Some requests might contain private data which should not be saved to a traffic log. Parameters which match the Matching Value and for which Mask is set will be masked / hashed when they are written to the logs.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| **Exclude WAF Rule** | <p>A parameter which fails its <strong>Matching Value</strong> comparison, and for which <strong>Restrict?</strong> is not selected, will be filtered by all WAF Rules except the ones whose IDs are listed here. (For example, some WAF Rules filter out special characters; if a certain parameter can legitimately contain these characters, it would make sense to exempt that parameter from those specific filters .) <br><br>It is also possible to always exempt a parameter from specific WAF Rules:</p><ul><li>List the rule IDs here</li><li>Ensure that <strong>Restrict?</strong> is not selected</li><li>And specify a <strong>Matching Value</strong> regex pattern that the parameter would never match.</li></ul> |

Constraints are defined in the same way for **Headers**, **Cookies**, and **Arguments** within their respective tabs.
