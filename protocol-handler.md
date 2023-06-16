# protocol-handler Well-Known Resource Identifier

URI suffix: protocol-handler  
Change controller: Fedi-To https://fedi-to.net/  
Specification document(s): https://github.com/fedi-to/fedi-to.github.io/blob/main/protocol-handler.md

## Purpose

The `protocol-handler` well-known resource endpoint is used to handle URLs in
a web-domain-specific way. (For the definition of "URL", consult Appendix A.)

It is supported for the `https` scheme and is intended to enable easier
interactions between webapps.

## Functional Description

The `protocol-handler` endpoint MUST be navigable, meaning a browser can be
pointed at it, and it MUST accept a `target` query (`GET`) parameter. This query
parameter contains the target URL that should be opened. This target URL MUST
be a full URL, including the scheme, encoded using the
[component percent-encode set]. Effectively, the endpoint MUST support the
[HTML #protocol-handler-invocation] procedure with an URL of
`https://yourdomain.example/.well-known/protocol-handler?target=%s` (and indeed,
this was chosen for compatibility with `registerProtocolHandler`), however it
places no restrictions on allowed schemes - in particular, `https` is also
acceptable.

When navigated to, this endpoint SHOULD appropriately handle the given target
URL. For optimal compatibility, it SHOULD return `400 Bad Request` for unknown
or unsupported schemes and it SHOULD return `307 Temporary Redirect` or
`308 Permanent Redirect` for schemes it can handle. The target URL SHALL be
handled in an app-specific way.

[component percent-encode set]: https://url.spec.whatwg.org/#component-percent-encode-set
[HTML #protocol-handler-invocation]: https://html.spec.whatwg.org/multipage/system-state.html#protocol-handler-invocation

## Security Considerations

The `protocol-handler` endpoint is a simple navigable web resource with a query
parameter, comparable to any other. However, care should be taken not to
implement it as an open redirect - if the protocol handler at example.org is
passed an URL with an example.net authority, it SHOULD NOT redirect to
example.net, but instead to an app in example.org that fetches the target URL
and provides the relevant content, with the appropriate security measures as
required by the app.

It is further worth stressing the importance of not using URLs to represent
actions, or allowing URLs to trigger actions unprompted: Any website can trigger
a `GET` request; allowing unprompted actions would enable a malicious website to
e.g. spread misinformation or malware.

# Appendix A - on the use of "URL"

The term "URL" refers to the concept of URL as defined in the [WHATWG URL spec].

It should not be confused with "URI". This is a deliberate choice because
browsers follow the WHATWG URL spec for parsing e.g. the `href` attribute in an
HTML `<a>` element, and this spec needs to be consistent with what browsers do,
for compatibility with `registerProtocolHandler`.

[WHATWG URL spec]: https://url.spec.whatwg.org/
