# The Challenge Process

## Detecting Bots, Verifying Humans

Much of the Curiefense interface focuses on configuration for:

* The detection and mitigation of hostile requests.
* The detection and mitigation of hostile behavior of requestors.

The **challenge process** is different. It is built into Curiefense, and provides a third approach to security. It mitigates threats based on the requestor's identity, environment, and behavior.

{% hint style="info" %}
Bot challenges are included in the Premium version of Curiefense.
{% endhint %}

When Curiefense receives the first request from a previously unknown traffic source \(below described as the "user"\) for a browser-based web application, this is the typical process that is followed.

1. **Curiefense challenges the user's browsing environment.** Curiefense uses a variety of proprietary, multi-faceted techniques to verify that this requestor is a human using a browser, instead of a bot using a headless browser or emulator.
   * **The user’s browser is subjected to several dozen tests**, verifying the features known to be supported by that browser. This includes hidden canvases, video and audio in various formats, WebRTC and other advanced networking protocols, screen resolution, and more.
   * **The browser is subjected to an invisible “attack”**: subtle errors are injected into the environment, and the browser engine’s reactions are captured and analyzed. Curiefense verifies that the exceptions and error messages are those which should be generated, if the browser is what it claims to be. \(It is very difficult for threat actors to spoof this behavior using headless browsers and emulators, since there is an infinite number of possible errors to which any browser can be subjected, and threat actors need to replicate the actual reactions for each possible input.\) 
   * **The above process only takes milliseconds**, and it is completely transparent to the end user.
   * **Once a browser has passed these tests, Curiefense signs it cryptographically.** This signature accompanies all subsequent activity.
2. **If the challenge is not passed, the request is suspected to be a bot, and another challenge is issued.** This process continues until a challenge is passed, or a threshold is reached \(e.g., via a Rate Limit\) to ban the requestor. 
3. **If the challenge is passed, the browser's session is authenticated**, and the browser receives cookies from Curiefense.
4. **The browser then automatically resubmits the original request**, but this time, the cookies are included. The user is granted access to the requested URL, resources, etc.
5. **Subsequent requests will also include the cookies,** and thus, they are not challenged.

This process happens quickly \(in a few milliseconds\), and is **invisible** to the user.  

## Active Challenges versus Passive Challenges

The process described above is the **active** challenge process. Out of the box, this is the challenge process that Curiefense uses.

{% hint style="info" %}
**We recommend that whenever possible, users also enable** _**passive**_ **challenges.** 
{% endhint %}

Passive challenges add three additional benefits:

* **They enable biometric behavioral profiling**: a much more powerful means of identifying automated traffic. More on this below.
* **In some situations, active challenges can interfere with certain metrics** such as those provided by Google Analytics. \(The initial referrer information is lost.\) If this is a problem, active challenges can be disabled. In this situation, passive challenges can provide effective bot protection instead. 
* **When caching is being done by a CDN**, active challenges will not occur for pages being served from the cache. Passive challenges are necessary for Curiefense to perform bot detection in this situation.

{% hint style="info" %}
**If possible, we recommend that users use both active and passive challenges.**
{% endhint %}

## Biometric Behavioral Profiling

With Biometric Bot Detection, Curiefense continually gathers and analyzes stats such as client-side I/O events, triggered by the user’s keyboard, mouse, scroll, touch, zoom, device orientation, movements, and more. Based on these metrics, the platform uses Machine Learning to construct and maintain behavioral profiles of legitimate human visitors. Curiefense learns and understands how actual humans interact with the applications and services it is protecting. Continuous multivariate analysis verifies that each user is indeed conforming to expected behavioral patterns, and is thus a human user with legitimate intentions. 

Using this approach, Curiefense bot detection accuracy is not only high, it is also robust and resistant to reverse-engineering by threat actors. Behavioral profiles are constructed based on private analytics data, and threat actors have no realistic way of obtaining this information.

{% hint style="info" %}
This is an important reason why we recommend that all users enable Passive Challenges if it is possible to do so. Biometric Behavioral Profiling provides much more robust detection of automated traffic than is possible without it.
{% endhint %}

## Implementation

Implementing Passive Challenges is simple. Place this Javascript code within the pages of your web applications:

```text
<script src="/c3650cdf-216a-4ba2-80b0-9d6c540b105e58d2670b-ea0f-484e-b88c-0e2c1499ec9bd71e4b42-8570-44e3-89b6-845326fa43b6" type="text/javascript"></script>
```

{% hint style="info" %}
The code snippet can go into the header or at the end of the page. **The best practice is to place it within the header.** This ensures that subsequent calls contain authenticating cookies.

\(This matters for the first page served to a visitor. Subsequent pages will already have the authenticating cookies to include with their requests.\)
{% endhint %}

{% hint style="info" %}
**Most users set up the code snippet as a tag within their tag manager.** This makes it simple to install it instantly across their entire site/application.
{% endhint %}

If desired, the script code can include `async` and/or `defer` attributes:

```text
<script async src="/c3650cdf-216a-4ba2-80b0-9d6c540b105e58d2670b-ea0f-484e-b88c-0e2c1499ec9bd71e4b42-8570-44e3-89b6-845326fa43b6" type="text/javascript"></script>
```

```text
<script defer src="/c3650cdf-216a-4ba2-80b0-9d6c540b105e58d2670b-ea0f-484e-b88c-0e2c1499ec9bd71e4b42-8570-44e3-89b6-845326fa43b6" type="text/javascript"></script>
```

These usually are not necessary, and their effect will depend on the placement of the script within the page. Their use is left to your discretion.

## Testing

To test the implementation, use a browser to visit a page containing the Javascript snippet. Once it runs, the browser should have a cookie named **rbzid**.

## Disabling Active Challenges \(only if necessary\)

There are two primary situations where users sometimes want to disable Active Challenges:

* **When a user needs site analytics** to correctly reflect all referrers. \(Active Challenges can interfere with this.\)
* **For API endpoints**. Active Challenges are designed to verify the client's browser environment; for most API calls, there is no browser environment to verify.

Other than those situations, Active Challenges can be very beneficial.

{% hint style="info" %}
**We recommend that you keep Active Challenges enabled if possible.** They automatically eliminate almost all DDoS traffic, scanning tools, and other hostile bot traffic.
{% endhint %}

If you wish to turn off Active Challenges, remove all tags from the "Deny Bot" column within [ACL Profiles](../settings/policies-rules/acl-policies.md).

{% hint style="danger" %}
**If you have not enabled Passive Challenges** \(and successfully tested them\), disabling Active Challenges is not recommended.
{% endhint %}



