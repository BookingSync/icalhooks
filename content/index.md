---
title: Makes iCal content updated after seconds, not hours
subtitle: iCal Hooks is a lightweight subscription layer on top of iCal.
---

**Status**: This document is a first draft and made to [open discussions](https://github.com/BookingSync/icalhooks) to improve calendar update time and server load.

# What is iCal Hooks?

iCal Hooks is allowing to notify a consumer when something changed, it's webhooks on top of iCal. Here a tiny call to a given URL to notify that the iCal content should be refetched.

# How does it works?

1) It's based on iCal and **fully backward compatible**.

2) The consumer includes a **http header** with a **hooks url** when pulling the iCal feed.

3) The provider **store this hooks url** and **send a request** to it the next time a change is made on it's associated iCal feed.

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

**Provider** receice the request from Consumer and:

a) Stores the iCal Hooks URL, certainly in database. `http://consumer.com/icalhooks` in our example.

b) Responds as expected by returning the iCal feed.

## Step 3

An update on **Provider** happen, Provider will then make a `HEAD` request to the iCal Hooks URL stored in Step 2.

Example query:

~~~
HEAD http://consumer.com/icalhooks HTTP/1.1
<blank line>
<blank line>
~~~

# Use cases

## For vacation rentals

Current iCal integrations mostly pull iCal updates every 6 to 12 hours. The risk of double bookings is then consireable and makes current integrations dangerous when used with instant bookings.

iCal Hooks makes availability updated in couple seconds which makes instant bookings safe (a risk is still present but reduced to a really low level).

# Security

The current design without any limitation on which consumer subscribe to a hook
going to which targeted domain could lead to DDOS attacks from the provider to a selected target.

Various authentication solutions can be discussed to mitigate this risk and at least know which consumer is potentially at the origin of such abuse.
