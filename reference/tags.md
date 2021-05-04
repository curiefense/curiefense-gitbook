# Tags

When an incoming request is received, Curiefense generates internal tags and assigns them to it.

Some tags are assigned early, and are used to make decisions about how the request is handled. For example, if a request's IP is found on the Spamhaus DROP list, it might be assigned a tag of "spamhaus". Then an [ACL Profile](../console/document-editor/acl-profiles.md) might block the request because it contains that tag.

Some tags can be generated during processing; they reflect the decisions that were made. For example, a request that was blocked because it violated a [Rate Limit](../console/document-editor/rate-limits.md) will be assigned a tag containing that Rate Limit's name. The tag will be shown in the [Access Log](../analytics/access-log.md).

Some tags are defined by the user, while others are generated automatically by Curiefense.

## User-Defined Tags

[Profiling Lists](../console/document-editor/profiling-lists.md) are user-defined lists of criteria with one or more associated tags. Requests which match the criteria will be assigned those tags.

A Profiling List can be based on an external list \(e.g., the Spamhaus DROP list\), or a user-defined custom list \(e.g., a list of IP addresses used by the internal QA team\).

## Automatic Tags

Many tags are generated automatically by Curiefense. Examples:

* Every request receives a tag of "all".
* Every request receives several tags according to its source \(the IP address, geolocation, etc.\)
* Every request receives two tags for the [URL Map](../console/document-editor/url-maps.md) that it matched \(both at the domain level and at the Path Map level\).
* Requests which violate a security policy or have other problems, receive tags with descriptive names \(e.g., the name of the policy that was violated\).

When tag names are generated from underlying values \(IP addresses, security rule names, etc.\), hyphens will replace spaces and special characters.

### Examples of automatic tags

| Value or Situation | Example tag |
| :--- | :--- |
| \(Every request\) | all |
| IP address | ip-192-168-0-2 |
| ASN | asn-1680 |
| Country | geo-canada |
| URL Map that the request matched | urlmap-default-entry |
| WAF signature \#100039 was violated. | waf-sig-100039 |
| A Rate Limit \(named "Rate limit rule 5/60"\) is being evaluated. | rate-limit-rule-5-60 |

Sometimes a request will get two separate tags that seem to be redundant. For example:

* `urlmap-default-entry`
* `urlmap-entry-default`

When a URL Map is matched with a request, a tag is generated for the URL Map itself, and for the Path Map that was used. If the names are similar \(which is true for default values, as in the example\), then the tags can appear to be redundant.

