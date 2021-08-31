# Multi-Stage Traffic Filtering

![](../.gitbook/assets/traffic-flow-v1.3.7%20%282%29.png)

Curiefense evaluates incoming traffic in a multi-stage filtering process. An HTTP/S request which passes all security tests will be forwarded to the backend. 

When constructing its security posture, it can be helpful to understand the filtering stages. They are:

* Global Filters: Curiefense assigns [automatically-generated tags](tags.md#automatic-tags) and [user-defined tags](tags.md#user-defined-tags) to the requests. During this process, requests will be blocked immediately if they match a [Global Filters List that has a blocking action](../settings/policies-rules/global-filters.md#action) defined for it.
* [Session Flow Control](../settings/policies-rules/flow-control.md) enforcement.
* \*\*\*\*[Rate Limit](../settings/policies-rules/rate-limits.md) enforcement.
* \*\*\*\*[ACL Policies](../settings/policies-rules/acl-profiles.md) enforcement.
* \*\*\*\*[WAF Policies](../settings/policies-rules/waf-policies.md) and [WAF Rules](../settings/policies-rules/waf-rules.md) enforcement.
* Transmission of request to the backend.

## 



