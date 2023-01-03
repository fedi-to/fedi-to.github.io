# Fedi-To Design Choices

## The registration flow

The registration flow is actually pretty strict:

- It requires navigation. It would technically be possible to make registration
    support iframes but by requiring navigation it takes away control from the
    page and gives it entirely to the user. This prevents many kinds of attacks.
- It also attempts to mimic the OAuth flow of other services, which brings
    familiarity with analogous systems. This further prevents more kinds of
    attacks.

Good UX for protocol handlers requires at least these 2 design choices.

## The fallback protocol handler (well-known resource identifier)

(Note: The technical spec is at [protocol-handler.md],
this is just design rationale and details specific to Fedi-To.)

The fallback protocol handler can be derived entirely from the URL, without the
need for any web requests. This avoids SSRF, XSS, and the legal liabilities of
making arbitrary web requests from a server. (This also means we cannot rename
the endpoint, should it become widespread, so we need to get this right on the
first try.)

The Fedi-To implementation is documented in [protocol-handler.md], Appendix A.

The one quirk is that Fedi-To assumes the passed in `target` is already
normalized, and makes no attempt to re-normalized it, but when Fedi-To URLs are
manually crafted, they might not be. We note that deployments already need to
handle the case of manually-crafted URLs since the endpoints are freely
navigable, so this isn't much of an issue.

[protocol-handler.md]: /protocol-handler.md
