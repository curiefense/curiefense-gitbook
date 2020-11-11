# Multi-Stage Traffic Filtering

![](../.gitbook/assets/traffic-cf-v1.0%20%281%29.png)

Curiefense evaluates incoming traffic in a multi-stage filtering process. An HTTP/S request which passes all security tests will be forwarded to the backend. 

When constructing its security posture, it can be helpful to understand the filtering stages. They are:

* Profiling: Curiefense assigns [automatically-generated tags](tags.md#automatic-tags) and [user-defined tags](tags.md#user-defined-tags) to the requests.
* \*\*\*\*[Rate Limit](../console/document-editor/rate-limits.md) enforcement.
* \*\*\*\*[ACL Profile](../console/document-editor/acl-profiles.md) enforcement.
* \*\*\*\*[WAF Profiles](../console/document-editor/waf-profiles.md) and [WAF Signatures](../console/document-editor/waf-signatures.md) enforcement.
* Transmission of request to the backend.

## 



