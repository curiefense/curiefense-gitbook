---
description: >-
  All notable changes to this project will be documented in this file. This
  project adheres to Semantic Versioning.
---

# Release Notes

## Version 1.5.0

### Added

* New multiple layers rate limit
* New content filter Active/Report/Ignored
* Content filter category and risk level added 
* Content type validation
* Region and subregion support
* When multiple security policies match an url, the longest match string (more specific) is selected


### Updated

* build-docker-images.sh fails to update on macOS

### Improved

* Rewrite of the default policy
* Rate limit can be based on the tag list
* Support for inverted regexp in matching
* Argument masking
* eu field in logging

### Removed

-

### Fixed

* [ui] Tag Rules adds an empty tag to each request
* [ui] Tag rules - we do not require at lease one tag
* [ui] Adding a third entry in Flow control presents an error
* [ui] Tag rule lists tags - we create tags twice for the same list
* [ui] Flow control - When creating a new sequence - we have only one section instead of two
* ACL profiles - when tags at "deny bot" and "deny" columns, the evaluation flow is not as described at manual
* Global filters - Error at proxy log when we add list of ips from http source without comment
* Rate limits - "Event" by:Header/Cookie/Argument block, even when we don't pass this Header/Cookie/Argument in request
* Flow control - "Count by" attribute:tag doesn't work
* Logging failure when source IP is from an EU country
* Security policy second added rate limit is not enforced
* Tests fails because of invalid dependency in curieconfctl
* Rate limit - with Threshold = 0 does't added the tag to tags list of kibana logs
* Flow control "Count by" - We do not count by the selected attribute
* Rate limit\Tag rules 503 response code blocks with 403
* Rate Limits with Ban Action does not unlock a blocking at the end of blocking time
* Tags - after deleting lists and not using the tags in ACL anymore we still present the tags
* Make request.attributes consistent
* Disable all tag rules except API discovery by default
* Policies & Rules Search - ACLs are not listed
* Mask PII data
* ACL Policies Deny Bot and Allow Bot are both being checked when ACL is not active
* Rate Limits with Redirect Action does not work as expected
* If several decisions are reached during the rate limiting or flow control phase, the strongest one is chosen. Previously, an arbitrary decision was selected bug
* Empty regex no longer match any values, preventing content filter bypasses
* Logging messages related to ACL blocks is now more informative
* Non existing selectors for rate limiting / flow control will now cause the request to not be processed by the relevant rule, instead of being bundled in a "no selector" group


### Enhanced

* [ui] Rate Limit - Include/Exclude should be changed to accept tags only
* [ui] Version Control - Add an option to undo a version revert
* [ui] Remove the blue top border from the header
* [ui] Toast status messages change
* Flow control, If 2 rules share the same last request, action initiated will be according to the hierarchy
* Add 'authority' to 'request' in log structure
* Implement a syslog input for curielogger 

