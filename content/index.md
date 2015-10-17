---
title: Makes iCal content updated after seconds, not hours
subtitle: iCal Hooks is a lightweight subscription layer on top of iCal.
---

**Status**: This document is a first draft and made to [open discussions](https://github.com/BookingSync/icalhooks) to improve calendar update time and server load.

# Motivation

Todays consumers are increasingly expecting instant on-demand experiences. Whether scheduling appointments or booking vacation rentals, fast & accurate information results in more successful consumer interactions. Stale information, such as calendar data, only results in lost opppertunties.

... an example ... 6 hour delay...

Today, iCal consumption is primarily pull based. Consumers have a choice: pull frequently which reduces the risk of stale data but comes at a higher resource cost or pull less frequently, but serve more stale data.

# Goals

* Mitigate stale data, which results in degraded user experiences
* continue to use (and work with) existing standards (iCal)
* minimize adoption costs
* minimize resource costs

# Proposed Solution

As an optional addition to the existing pull based iCal flow, *iCal hooks* proposes a mechanism by which a consumer can register interest in a future change to the given requested resource.

Supporting iCal providers can opt to inform requesting subscribed consumer of changes. Doing so, enables a simple near-realtime unidirectional synchronization primitive.

# What iCal Hooks are not?

iCal Hooks does not solve bi-directional synchronization. Although also valuable, we consider this out-of-scope and recommend those interested in pursuing an alternatives.

# How does it work?

1) It's based on iCal and **fully backward compatible**.
2) A consumer includes a **http header** with a **hook url** when issueing a `GET` request to a iCal feed.
3) A provider **stores the provided hook url** as a subcription to the given resource.
4) When the iCal resource changes, the provider removes the earlier **hook url** and sends a `HEAD` request to the earlier provided hook URL. Informing the consumer that its data is most likely stale.
5) The consumer issues another `GET` request, and if it wishes to once again subscribe the corresponding hook url.

*note: If the consumer wants to **unsubscribe**, it just doesn't include the HTTP header on future `GET`'s*

# Example

**Provider** is the originating server and exposes an iCal feed at `http://provider.com/ical.ics`
**Consumer** is a client subscribing to `http://provider.com/ical.ics`

## Step 1

**Consumer** make a GET request to `http://provider.com/ical.ics` including a `X-ICALHOOKS-URL` HTTP Header.

Sample Request:

~~~
GET http://provider.com/ical.ics HTTP/1.1
X-ICALHOOKS-URL: http://consumer.com/icalhooks
<blank line>
<blank line>
~~~

## Step 2

**Provider** receive the request from Consumer and:

1) Store the iCal Hooks URL, `http://consumer.com/icalhooks` in our example.

2) Respond as expected by returning the iCal feed.

## Step 3

An update on **Provider** occurs, the provider will:

1) make a `HEAD` request to the iCal Hooks URL stored in Step 2.

2) remove this URL from it's storage.

Example query:

~~~
HEAD http://consumer.com/icalhooks HTTP/1.1
<blank line>
<blank line>
~~~

# Security

The current design can allow a malicious consumer to make the provider send `HEAD` requests to a targeted URL. Rate limiting can be added by the provider to mitigate such risk.

Adding authentication on the subscription could be added but the current design makes it really easy to make this specification adopted with a known risk being low and mitigeable. If we realize that abuse are made, we can improve this specification design and add an extra authentication layer but the cost of developement and ease of adoption would be dramatically impacted.
