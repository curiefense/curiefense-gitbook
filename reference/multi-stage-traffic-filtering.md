# Multi-Stage Traffic Filtering

![](../.gitbook/assets/traffic-cf-v1.0%20%281%29.png)

Curiefense evaluates incoming traffic in a multi-stage filtering process. An HTTP/S request which passes all security tests will be forwarded to the backend. 

When constructing its security posture, it can be helpful to understand the filtering stages. They are:

* Tagging: Curiefense assigns [automatically-generated tags](tags.md#automatic-tags) and [user-defined tags](tags.md#user-defined-tags) to the requests.
* \*\*\*\*[Rate Limit](../settings/policies-rules/rate-limits.md) enforcement.
* \*\*\*\*[ACL Policies](../settings/policies-rules/acl-policies.md) enforcement.
* \*\*\*\*[WAF Policies](../settings/policies-rules/waf-policies.md) and [WAF Rules](../settings/policies-rules/waf-rules.md) enforcement.
* Transmission of request to the backend.

## 



