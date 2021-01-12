# Explainer - Partition Network State
Author: mmenke@google.com
Last update:  Jan 11, 2021

## Introduction

A browser's network resources, such as connections, DNS cache, and alternative service data are generally shared globally.  That is, requests to the same destination across pages can reuse the same socket, and DNS lookups across pages will use the same cache.  This allows for side-channel timing attacks, where one site can figure out if another has been visited recently. For example, if the connection is made quickly, it may be assumed that the socket was warm. It also allows for third parties to track users across first party contexts they are loaded in using a variety of techniques (tracking socket reuse, using per-user alternative service advertisements, etc).  See Chrome [Privacy Sandbox](https://github.com/michaelkleber/privacy-model) privacy model for more information.

We propose to [partition](https://github.com/michaelkleber/privacy-model#identity-is-partitioned-by-first-party-site) much of this state to prevent these resources from being shared across first party contexts to protect against these sorts of attacks.  To do this, each request will have an additional "network partition key" that must match in order for resources to be reused.

This extra key will necessarily make third party resources less reusable, as sites will not be able to access shared resources and metadata learned from loading other sites.

## Network Partition Key

This is covered by the earlier [HTTP cache partitioning explainer](https://github.com/shivanigithub/http-cache-partitioning/) as well as the [fetch spec](https://fetch.spec.whatwg.org/#network-partition-key). We propose to use the two value (top-level site, iframe site) key described in the HTTP cache partitioning explainer as "triple keying".

A "transient network partition key" is a network partition key for an [opaque origin](https://html.spec.whatwg.org/multipage/origin.html#concept-origin-opaque), and stores no data on disk. In several cases mentioned below, such keys are created for use by internal network requests.

## Proposed solution

We propose to use the network partition key to partition connections and certain other network information, and only use state with a matching key for requests, in addition to whatever the object was previously keyed on. This will increase the eviction rate of various object types, since resource limitations require there be limits on the amount stored network objects. These limits do potentially leak information across network partition keys when evicting data, but addressing that is beyond the scope of this proposal.

### What resources will and will not be keyed on network partition key:

We propose to key the following connection-associated resources using the network partition key:

* Live DNS requests and the DNS cache
* HTTP/1.x sockets
* HTTP/2 and HTTP/3 sessions
* WebSockets over shared HTTP/2 and HTTP/3 sessions
* Connections to proxies (though the result of DNS lookups for the hostnames of proxy servers will be shared globally)
* The TLS and HTTP/3 session resumption caches
* Alternative service information, including information about which servers have broken alternative services
* Cache of which servers support HTTP/2 (used to avoid creating extra sockets when establishing a connection to HTTP/2 compatible servers)
* DNS lookups from PAC scripts will use the network partition key of the request causing the PAC script to issue the lookup
* Cached information from Expect-CT headers.

This particular proposal does not cover some other types of network information. For clarity, here are some types of network information and resources not covered by this proposal, though this is likely not a complete list.

These objects will likely need to respect the network partition key in the future, but will need to be covered by other explainers and spec work:

* HTTP cache [already has an explainer of its own](https://github.com/shivanigithub/http-cache-partitioning/blob/master/README.md).
* HTTP auth cache
* Client certs
* Clear-Site-Data header

Other solutions will likely need to be used for these:

* [Cookies](https://blog.chromium.org/2020/01/building-more-private-web-path-towards.html)
* HSTS cache
* Cert validation (Both the verification cache itself, and OCSP/CRL/ACA network fetches, which will use a single transient network partition key for now, at least)
* Reporting API. The latest (draft of the spec)[https://w3c.github.io/reporting/] addresses tracking concerns, by making it document-scoped.
* Network error logging

Some potential cross-site information leaks involve upstream resources that are not under the control of a browser, and others involve trusted third parties and would have a significant performance impact to mitigate.  Here are some other potential leaks not currently covered by this explainer, with no work currently planned to address them:
* The browser can't do much about upstream or OS-layer DNS caches, though using DoH without fallback or the built-in DNS resolver will bypass OS caches, at least.
* While DNS resolutions will respect the key, and live lookups will not be merged across network partition keys, partitioning DoH HTTPS requests themselves may not be worth the performance cost.
* Loading a single instance of the configured PAC script would likely require too much memory, and put too much load on servers hosting PAC files.
* PAC script fetches delegated to the OS cannot be keyed on the network partition key.
* On platforms where certificate validation is deferred to the OS, the OS itself will likely cache any data it needs to fetch from the network (e.g. for revocation checks or AIA fetches).

## Performance 

We expect partitioning network state to cause a modest reduction in performance, since connections can no longer be shared with cross-site iframes, or across different top-level sites.  It’s also less likely that we’ll have HTTPS/QUIC session resumption information for a network partition, and we’ll be less likely to use QUIC for the initial connection to a site.

Our current experiment is small, and shows a performance change that’s generally within the margin of error.  We’ll release numbers once the experiment is larger, and we have more confidence in the accuracy of our numbers.

## Acknowledgements
This explainer takes heavy inspiration from Shivani Sharma’s [HTTP cache partitioning explainer](https://github.com/shivanigithub/http-cache-partitioning/).  It also reflects extensive feedback from Josh Karlin and Paul Jensen.
