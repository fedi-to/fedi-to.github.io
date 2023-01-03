# protocol-handler Well-Known Resource Identifier

URI suffix: protocol-handler  
Change controller: Fedi-To https://fedi-to.net/  
Specification document(s): https://github.com/fedi-to/fedi-to.github.io/blob/main/protocol-handler.md

## Purpose

The `protocol-handler` well-known resource endpoint is used to handle Web-First
Protocol Handlers, as implemented by Fedi-To. This allows Fedi-To to support
arbitrary fallback handlers instead of requiring the fallback to be one of the
built-in handlers, allowing new instances to be launched that can be handled by
existing built-in handlers, while providing a fallback to that instance of the
service.

## Functional Description

The `protocol-handler` endpoint MUST be navigable, meaning a browser can be
pointed at it, and it MUST accept a `target` query parameter. This query
parameter contains the target URL that should be opened. This target URL MUST
be a full URL, including the scheme, encoded using the
[component percent-encode set]. Effectively, the endpoint is used following the
steps in [HTML #protocol-handler-invocation] \(as of 2022-12-26), as if the
registered protocol handler were for `/.well-known/protocol-handler?target=%s`,
but skipping steps 2 and 3. (Indeed, this was chosen for compatibility with
`registerProtocolHandler`.)

When navigated to, this endpoint SHOULD appropriately handle the given target
URL. For optimal compatibility, it SHOULD return `404 Not Found` for unknown or
unsupported schemes and it SHOULD return `307 Temporary Redirect` or
`308 Permanent Redirect` for schemes it can handle.

[component percent-encode set](https://url.spec.whatwg.org/#component-percent-encode-set)
[HTML #protocol-handler-invocation](https://html.spec.whatwg.org/multipage/system-state.html#protocol-handler-invocation)

## Security Considerations

The `protocol-handler` endpoint is a simple navigable web resource with a query
parameter, comparable to any other. However, care should be taken not to
implement it as an open redirect - if the protocol handler at example.org is
passed an URL with an example.net authority, it SHOULD NOT redirect to
example.net, but instead to an app in example.org that fetches the target URL
and provides the relevant content, with the appropriate security measures as
required by the app.
