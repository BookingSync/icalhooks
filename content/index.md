---
title: Makes iCal content updated after seconds, not hours
subtitle: iCal Hooks is a lightweight subscription layer on top of iCal.
---

**Status**: This document is a first draft and made to [open discussions](https://github.com/BookingSync/icalhooks) to improve calendar update time and server load.

# Motivation

Their's a common trend to develop real-time applications, users don't like stale data and latency as they degrade their experience. Whether it's during a vacation rental booking process, where availability needs to be as real-time as possible to do instant booking or when connecting events or scheduling systems together like Google Calendar and your appointment software.

# The goal

The goal is to add the smallest layer possible to bring real-time support on top of current iCal technologies and with the minimum cost.

# What is iCal Hooks?

iCal Hooks allow a iCal consumer to subscribe to changes so he's notified in near real-time. It's webhooks with a really simple subscription logic.

# What is not iCal Hooks?

iCal Hooks does not solve synchronization issues but lower the latency for a provider to send changes to a consumer service. It then reduce conflicts and provide near real-time updates and can also lower server load as you don't need high frequency pulls any longer.

# How does it works?

1) It's based on iCal and **fully backward compatible**.

2) The consumer includes a **http header** with a **hooks url** when pulling the iCal feed.

3) The provider **store this hooks url** and **send a request** to it the next time a change is made on it's associated iCal feed. This URL is then removed as meant to be a one time subscription. This help handling throttling.

4) If the consumer wants to unsubscribe, it just don't include the HTTP `header` on future pulls.

# Example

**Provider** is the originating server and exposes an iCal feed at `http://provider.com/ical.ics`

**Consumer** is a client subscribing to `http://provider.com/ical.ics`

## Step 1

**Consumer** make a GET request to `http://provider.com/ical.ics` including a `X-ICALHOOKS-URL` HTTP Header.

Sample query:

~~~
GET http://provider.com/ical.ics HTTP/1.1
X-ICALHOOKS-URL: http://consumer.com/icalhooks
<blank line>
<blank line>
~~~

## Step 2

**Provider** receive the request from Consumer and:

a) Store the iCal Hooks URL, `http://consumer.com/icalhooks` in our example.

b) Respond as expected by returning the iCal feed.

## Step 3

An update on **Provider** happen, Provider will then make a `HEAD` request to the iCal Hooks URL stored in Step 2 and remove this URL from it's storage.

Example query:

~~~
HEAD http://consumer.com/icalhooks HTTP/1.1
<blank line>
<blank line>
~~~

# Security

The current design can allow a malicious consumer to make the provider send `HEAD` requests to a targeted URL. Rate limiting can be added by the provider to mitigate such risk.
Adding authentication on the subscription could be added but the current design makes it really easy to make this specification adopted with a known risk being low and mitigeable. If we realize that abuse are made, we can improve this specification design and add an extra authentication layer but the cost of developement and ease of adoption would be dramatically impacted.
