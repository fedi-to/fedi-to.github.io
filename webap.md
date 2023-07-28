# The `web+ap` URI

The `web+ap` URI scheme is intended to represent web-based ActivityPub objects,
as seen in the [ActivityPub specification].

## Syntax

Same as `https`, with the following additional restrictions:

1. Userinfo is not allowed for this scheme, and if present, MUST be ignored.

See [RFC9110, Section 4.2.2](https://www.iana.org/go/rfc9110) for `https`.

## Encoding considerations

Same as `https`.

## Interoperability considerations

Currently, none of the ActivityPub implementations parse `web+ap` URIs. There
is an open redirector that parses `web+ap` URIs and turns them into the correct
`https` URI, on a best-effort basis.

It is worth noting that browsers parse these using the [WHATWG URL spec],
especially when used as e.g. a web-based protocol handler.

[WHATWG URL spec]: https://url.spec.whatwg.org/

## Functional description

A `web+ap` URI is an `https` URI with additional semantics. ActivityPub software
supporting this URI MUST treat it as an `https` URI and then follow the
[ActivityPub specification], especially when it comes to the exchanged content
types (the `Accept` and `Content-Type` header).

It is also encouraged for operating systems to provide a fallback mechanism for
these URIs. This SHOULD be done in a way that the target website can reject
non-ActivityPub objects (like invite links). This fallback mechanism SHOULD be
used when no appropriate ActivityPub software is available.

[ActivityPub specification]: https://www.w3.org/TR/activitypub/

## Security considerations

The security considerations are basically the same as `https`, with additional
considerations for this being a valid scheme for registering a [web-based
protocol handler]. The purpose of this URI is to explicitly allow registering
a web-based protocol handler, particularly one of the user's choice, for
interacting with the ActivityPub network. More specifically, this would allow
the user to use their own e.g. Mastodon instance to interact with ActivityPub
URIs, automatically.

No further security considerations are currently known.

[web-based protocol handler]: https://developer.mozilla.org/en-US/docs/Web/API/Navigator/registerProtocolHandler/Web-based_protocol_handlers
