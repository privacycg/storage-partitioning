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

There are many standards that together make up a user agent and many of these standards define “problematic” state. This repository’s [issue tracker](https://github.com/privacycg/storage-partitioning/issues) is where we're coordinating the effort to address these issues in an ideally holistic manner. The actual changes will happen in each impacted standard and are collated here for visibility.

## Additional keying

[whatwg/html #4966](https://github.com/whatwg/html/pull/4966) defined the [top-level origin](https://html.spec.whatwg.org/multipage/webappapis.html#top-level-origin) concept for an environment and HTML also defines site and [obtain a site](https://html.spec.whatwg.org/multipage/origin.html#obtain-a-site). Together these allow for a definition of top-level site, which most user agents are targeting as additional key.

For some user agent state it might be beneficial to add even more keys, e.g., to prevent attacks between framed documents. [shivanigithub/http-cache-partitioning #2](https://github.com/shivanigithub/http-cache-partitioning/issues/2) has some relevant discussion.

## Relaxing additional keying

For some user agent state (Cookies and Storage below in particular are under discussion) the additional keying might be relaxable. This is tracked by [The Storage Access API](https://privacycg.github.io/storage-access/) though also necessarily has to inform the work noted below as it might impact architecture decisions.

## Blocking

Aside from using additional keying, outright blocking of the user agent state is also considered at times, e.g., for Cookies or as happens today for Storage in opaque origins. This is not likely to be web compatible nor even desirable for all user agent state, but could well be a valid solution for some.

## User agent state

A likely inexhaustive enumeration of user agent state and ongoing standards activity:

* Cookies
* Network state:
   * HTTP cache ([whatwg/fetch #904](https://github.com/whatwg/fetch/issues/904), [whatwg/fetch #943](https://github.com/whatwg/fetch/pull/943))
   * Connections ([whatwg/fetch #917](https://github.com/whatwg/fetch/issues/917))
      * Also consider speculative connections
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
* Storage ([whatwg/storage #18](https://github.com/whatwg/storage/issues/18)):
   * Indexed DB
   * Cache API
   * `localStorage`
   * `sessionStorage`
   * `history.pushState()`
   * `new Notification()`
   * AppCache (actively being removed, probably not relevant)
* Storage (communication channels):
   * `BroadcastChannel`
   * Shared workers
   * Service workers
   * Web Locks
* Web Authentication
* WebRTC’s `deviceId` ([w3c/mediacapture-main #675](https://github.com/w3c/mediacapture-main/issues/675))
* Blob URL store ([w3c/FileAPI #153](https://github.com/w3c/FileAPI/issues/153))
* HTML Standard’s list of available images
* `window.name`
* Browsing context group's agent cluster map (only observable with popups)
* Permissions ([Feature Policy](https://w3c.github.io/webappsec-feature-policy/) allows these to be disabled by default when the top-level site is not equal to the current site and require explicit delegation in such cases)
   * Persistent storage ([whatwg/storage #87](https://github.com/whatwg/storage/issues/87))
* Non-standardized features:
   * Credentials (username and password storage)
   * Form autofill data storage
   * Per-site user preferences
   * Favicon cache
   * Page info media previews
   * Save Page As

If there is state or standards activity missing please [file an issue](https://github.com/privacycg/storage-partitioning/issues/new) or provide a pull request.

## Acknowledgments

The author of this document was inspired by Chromium’s Network Isolation Key, Firefox and Tor Browser’s First-Party Isolation, Safari’s Intelligent Tracking Prevention, XS-Leaks, and the many people wanting to improve these aspects of the web.
