# Client-Side Storage Partitioning

A [Work Item](https://privacycg.github.io/charter.html#work-items)
of the [Privacy Community Group](https://privacycg.github.io/).

## Participate
- [GitHub privacycg/storage-partitioning](https://github.com/privacycg/storage-partitioning) ([new issue](https://github.com/privacycg/storage-partitioning/issues/new), [open issues](https://github.com/privacycg/storage-partitioning/issues))
- [GitHub privacycg/storage-partitioning/commits](https://github.com/privacycg/storage-partitioning/commits)

## Introduction

User agent state that is keyed by a single [origin](https://html.spec.whatwg.org/multipage/origin.html#concept-origin) or [site](https://html.spec.whatwg.org/multipage/origin.html#site) is an acknowledged privacy and security bug. Through side-channels or more directly, this allows:

1. A top-level site `https://site-a.example` _A_ to infer that a user is also visiting top-level site `https://site-b.example` _B_, by embedding resources or documents from _B_ in _A_. Beyond visiting, it can also allow _A_ to infer specific state from _B_ that depends on the user, thereby revealing many aspects of the user. [Timing Attacks on Web Privacy](https://sip.cs.princeton.edu/pub/webtiming.pdf), [XS-Leaks](https://github.com/xsleaks/xsleaks), and [COSI](https://arxiv.org/pdf/1908.02204.pdf) discuss this in more detail.
2. Conversely, it allows a site `https://tracker.example` whose resources might be embedded on many different sites, to track the end user across these sites.

To solve a key aspect of this, any such user agent state needs to be keyed by more than a single origin or site.

There are many standards that together make up a user agent and many of these standards define “problematic” state. This repository’s [issue tracker](https://github.com/privacycg/storage-partitioning/issues) is where we're coordinating the effort to address these issues in a holistic manner. The actual changes will happen in each impacted standard and are collated here for visibility.

## Additional keying

[whatwg/html #4966](https://github.com/whatwg/html/pull/4966) defined the [top-level origin](https://html.spec.whatwg.org/multipage/webappapis.html#concept-environment-top-level-origin) concept for an environment and HTML also defines site and [obtain a site](https://html.spec.whatwg.org/multipage/origin.html#obtain-a-site). Together these allow for a definition of top-level site, which most user agents are targeting as additional key. When this additional key is cross-site from the “normal” key, the relevant state can be considered to be partitioned.

For some user agent state it might be beneficial to add even more keys, e.g., to prevent attacks between framed documents. [shivanigithub/http-cache-partitioning #2](https://github.com/shivanigithub/http-cache-partitioning/issues/2) has some relevant discussion.

## Relaxing additional keying

For some user agent state (Cookies and Storage below in particular are under discussion) the additional keying might be relaxable. This is tracked by [The Storage Access API](https://privacycg.github.io/storage-access/) though also necessarily has to inform the work noted below as it might impact architecture decisions.

## Blocking

Aside from using additional keying, outright blocking of the user agent state is also considered at times, e.g., for cross-site cookies or as happens today for storage in opaque origins. This is not likely to be web compatible nor even desirable for all user agent state, but could well be a valid solution for some.

## User agent state

This section contains a likely inexhaustive enumeration of user agent state and ongoing standards activity. If there is state or standards activity missing please [file an issue](https://github.com/privacycg/storage-partitioning/issues/new) or provide a pull request.

### Cookies

The tentative overall plan is to block cross-site cookies and add support for partitioned cookies via opt-in. The details are still under discussion though it seems likely to be eventually standardized across the IETF and WHATWG. Relevant discussions:

* [Cookie layering](https://github.com/httpwg/http-extensions/issues/2084) (a start of a discussion with the IETF how to best structure the HTTP State Management Mechanism specification to account for these changes to cookies).
* For opt-in partitioned cookies [CHIPS](https://github.com/WICG/CHIPS) is the most likely candidate, though a few favor [an approach using `requestStorageAccess()`](https://github.com/privacycg/storage-access/issues/75). These two approaches are not necessarily conflicting and some browsers have expressed interest in supporting both.
* Meetings: 
  * [Cross-site cookies standardization](https://github.com/privacycg/meetings/issues/16) ([minutes](https://github.com/privacycg/meetings/blob/main/2022/telcons/04-28-minutes.md)).
  * [Cross-site cookies standardization, part 2](https://github.com/privacycg/meetings/issues/19) ([minutes](https://github.com/privacycg/meetings/blob/main/2022/telcons/05-12-minutes.md)).

### Remaining user agent state

* Network state:
   * HTTP cache (standardized in Fetch)
   * HTTP connections (standardized in Fetch)
      * Also consider speculative connections (unclear where these are created in standards, but if done through Fetch it would be correct)
   * WebSocket connections ([whatwg/fetch #1243](https://github.com/whatwg/fetch/issues/1243))
   * WebRTC connections ([w3c/webrtc-pc #2613](https://github.com/w3c/webrtc-pc/issues/2613))
   * WebTransport connections ([w3c/webtransport #128](https://github.com/w3c/webtransport/issues/128))
   * DNS
   * HTTP authentication
   * Alt-Svc
   * Fonts
   * HSTS
   * TLS client certificates
   * TLS session identifiers
   * HPKP
   * OCSP
   * Intermediate CA cache
   * Prefetch
   * Preconnect
   * CORS-preflight cache (standardized in Fetch)
* Storage ([whatwg/storage #88](https://github.com/whatwg/storage/issues/88), [whatwg/storage #90](https://github.com/whatwg/storage/issues/90)):
   * Indexed DB
   * Cache API
   * `localStorage`
   * `sessionStorage`
   * `BroadcastChannel` ([whatwg/html #5803](https://github.com/whatwg/html/issues/5803))
   * Shared workers
   * Service workers
   * Web Locks
* Web Authentication
* WebRTC’s `deviceId` ([w3c/mediacapture-main #675](https://github.com/w3c/mediacapture-main/issues/675))
* Blob URL store ([w3c/FileAPI #153](https://github.com/w3c/FileAPI/issues/153))
* HTML Standard’s list of available images
* `window.name` (standardized in HTML)
* Browsing context group's agent cluster map (only observable with popups)
* Permissions ([Permissions Policy](https://w3c.github.io/webappsec-permissions-policy/) largely allows these to be disabled by default when the top-level site is not equal to the current site and require explicit delegation in such cases)
   * Persistent storage ([whatwg/storage #87](https://github.com/whatwg/storage/issues/87))
   * Notifications ([whatwg/notifications #177](https://github.com/whatwg/notifications/issues/177))
* WebGL and WebGPU's cache of compiled shaders and pipelines (standardized by highlighting the risk in the security/privacy consideration section as the caches are only observable through timing)
* Non-standardized features:
   * Credentials (username and password storage)
   * Form autofill data storage
   * Per-site user preferences
   * Favicon cache
   * Page info media previews
   * Save Page As

## Presentation

The author of this document gave a short presentation in early 2022 about the state of this effort: [State of state partitioning](https://docs.google.com/presentation/d/1i7KvTtIS2JhAadQsdWLFpMzNmgXmUbXSfPuO_wYX6d8/edit).

## Acknowledgments

The author of this document was inspired by Chromium’s Network Isolation Key, Firefox and Tor Browser’s First-Party Isolation, Safari’s Intelligent Tracking Prevention, XS-Leaks, and the many people wanting to improve these aspects of the web.
