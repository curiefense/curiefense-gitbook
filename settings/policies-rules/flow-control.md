# Flow Control

![](../../.gitbook/assets/session-flow-control%20%281%29.png)

The input controls at the top of this page are described here: [Policies & Rules Entry Administration](./#entry-administration). Specific editing of a Session Flow Control entry is described below.

## Overview

The Session Flow Control module blocks hostile activity based on defined sequences of session flow.

Threat actors usually behave quite differently than legitimate users. For speed and efficiency, they tend to deviate from normal patterns of activity. The Session Flow Control capabilities of Curiefense allow you to define the expected patterns of behavior, and block access attempts that deviate from them.

For example, when a legitimate user attempts to log into a web application, the initial access of the login page will generate a GET request. Subsequently, a POST request will arrive with the login credentials. 

However, a hostile bot that's attempting a credential stuffing attack has no need to issue a GET, and often, will not bother to do so. Therefore, if a POST request arrives that was not preceded by a GET, this is anomalous behavior, and Curiefense can block it.

## Flow Control Parameters

| Value | Description |
| :--- | :--- |
| **Name** | A name for this flow control entry, for display within the interface. |
| **Active** | Whether or not this flow control entry is enforced. |
| **TTL** | If a traffic source deviates from the **Flow Control Sequence** \(described below\), the TTL is the length of time that the **Action** will be performed. |
| **Count** **by** | Defines the criteria by which Curiefense will associate requests with a single requestor. In other words, this is how Curiefense identifies requests as having originated from the same traffic source. By default, a single parameter is available; to add more, select **New entry**. Multiple parameters are evaluated with "AND"; requests must match all the parameters to be associated together.  |
| **Action** | When the **Flow Control Sequence** is violated, this **Action** will be taken for the time period specified in **TTL**. |
| **Notes** | Comments for use within the interface. |
| **Include** | Includes all requests in the evaluation that contain one or more [Tags](../../reference/tags.md) on this list. |
| **Exclude** | Excludes any request from evaluation if it contains a Tag on this list. |

## Flow Control Sequence

These parameters define the sequence of requests that will be enforced. The sequence consists of several **sequence sections**. They must be fulfilled in the order defined here.

By default, a new sequence contains two sections. Additional sections can be added by selecting the "**Create new sequence section**" button.

### Parameters for each Sequence Section

A request will fulfill this Sequence Section if it matches all of these parameters:

* The HTTP method specified in **Method**
* The domain or host specified in **Host**
* The path specified in **Path**
* And the **optional parameters**, if any. Optional parameters can be added by selecting the "**+**" button; each parameter includes matching characteristics for a header, cookie, or argument.





