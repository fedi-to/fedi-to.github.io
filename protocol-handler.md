# protocol-handler Well-Known Resource Identifier

URI suffix: protocol-handler  
Change controller: Fedi-To https://fedi-to.net/  
Specification document(s): https://github.com/fedi-to/fedi-to.github.io/blob/main/protocol-handler.md

## Purpose

The `protocol-handler` well-known resource endpoint is used to handle Fedi-To
Web Protocol Handlers. This allows Fedi-To to support arbitrary fallback
handlers instead of requiring the fallback to be one of the built-in handlers,
allowing new instances to be launched that can be handled by existing built-in
handlers, while providing a fallback to that instance of the service.

## Functional Description

The `protocol-handler` endpoint MUST be navigable, meaning a browser can be
pointed at it, and it MUST accept a `target` query parameter. This query
parameter contains the target URL that should be opened. This target URL MUST
be a full URL, including the scheme.

When navigated to, this endpoint SHOULD appropriately handle the given target
URL. It is currently unspecified whether this should be done with a 3xx
redirect, javascript, or some other means.

## Security Considerations

The `protocol-handler` endpoint is a simple navigable web resource with a query
parameter, comparable to any other. However, care should be taken not to
implement it as an open redirect - if the protocol handler at example.org is
passed an URL with an example.net authority, it SHOULD NOT redirect to
example.net, but instead to an app in example.org that fetches the target URL
and provides the relevant content, with the appropriate security measures as
required by the app.
