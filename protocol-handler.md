# protocol-handler Well-Known Resource Identifier

URI suffix: protocol-handler  
Change controller: Fedi-To https://fedi-to.net/  
Specification document(s): https://github.com/fedi-to/fedi-to.github.io/blob/main/protocol-handler.md

## Purpose

The `protocol-handler` well-known resource endpoint is used to handle Web-First
Protocol Handlers. It allows Web-First Protocol URLs, that is, URLs with
schemes of the form `web+example:`, with no otherwise known handler, to be
successfully handled provided that the URL contains an appropriate authority
section. (For the definition of "URL", consult Appendix B.)

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

The authority section is parsed following the same [WHATWG URL spec] rules as
for `https` URLs, except it MUST strictly follow two forward slashes (`//`)
after the scheme section. That is, `web+example:\\example` does not have an
authority section, but `web+example://example\` does. The following table shows
some examples; note that all of these examples are valid URLs (they can be
handled by an explicit protocol handler), but only those with an authority can
be handled when no explicit protocol handler is known.

| URL | Authority |
| --- | --- |
| `web+example:///` | None |
| `web+example://\` | None |
| `web+example:///example` | None |
| `web+example:\\example` | None |
| `web+example://example.org\` | `example.org` |
| `web+example://example.org` | `example.org` |

For one possible implementation of this process, refer to Appendix A.

When navigated to, this endpoint SHOULD appropriately handle the given target
URL. For optimal compatibility, it SHOULD return `404 Not Found` for unknown or
unsupported schemes and it SHOULD return `307 Temporary Redirect` or
`308 Permanent Redirect` for schemes it can handle.

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

# Appendix A - Fedi-To implementation

The following Rust code shows how Fedi-To implements the application side of
this specification. It uses version `2.3.1` of the `url` crate.

```rust
use percent_encoding::utf8_percent_encode;

/// Error kind returned when trying to find the fallback protocol handler.
enum FallbackError {
    /// Returned when the given URL, while valid, does not provide a fallback
    /// handler.
    NoHandler,
    /// Returned when the given target is not an URL.
    NotAnUrl,
}

/// Checks whether the `scheme` part of `web+scheme` satisfies the desired
/// constraints.
fn is_scheme_invalid(scheme: &str) -> bool {
    // valid schemes are non-empty and are entirely ascii lowercase
    // so invalid schemes are empty or contain non-ascii-lowercase.
    scheme.is_empty() || !scheme.trim_start_matches(|c: char| -> bool {
        c.is_ascii_lowercase()
    }).is_empty()
}

/// Attempts to find a fallback protocol handler for the given target URL.
///
/// The target is assumed to be normalized, as per the WHATWG URL spec. (Note
/// that Fedi-To doesn't actually check that it is, but that's a Fedi-To
/// issue.)
fn get_fallback(target: &str) -> Result<String, FallbackError> {
    use FallbackError::*;
    // find the scheme
    let scheme = {
        let colon = target.find(':').ok_or(NotAnUrl)?;
        let scheme = &target[..colon];
        if !scheme.starts_with("web+") {
            return Err(NotAnUrl);
        }
        let scheme = &scheme[4..];
        if is_scheme_invalid(scheme) {
            return Err(NotAnUrl);
        }
        scheme
    };
    // replace web+scheme with https
    // this allows us to handle web+ URLs with the semantics we actually
    // want, which is roughly the same as https, with a few differences
    let mut as_if_https = target.to_string();
    as_if_https.replace_range(0..4+scheme.len(), "https");
    // the main difference is that unlike https, authority is optional.
    // so, first check that there should be an authority.
    if !as_if_https.starts_with("https://") {
        return Err(NoHandler);
    }
    // then also check that the authority actually exists.
    // this is necessary so we don't end up parsing web+example:///bar as
    // web+example://bar/ (which would be wrong).
    // note that we do parse web+example://bar\ as an authority! (but
    // everything else - like the path - we treat as opaque to us)
    if as_if_https.starts_with("https:///")
    || as_if_https.starts_with("https://\\") {
        return Err(NoHandler);
    }
    // NOTE: we only do this parse to extract the domain/port, it is up to
    // the protocol-handler to deal with malformed or malicious input.
    // NOTE: this is the same URL parser as used by browsers when handling
    // `href` so this is correct.
    let mut url = url::Url::parse(&*as_if_https).map_err(|_| NoHandler)?;
    url.set_path("/.well-known/protocol-handler");
    let _ = url.set_username("");
    let _ = url.set_password(None);
    let mut params = "target=".to_owned();
    params.extend(utf8_percent_encode(&*target, COMPONENT));
    url.set_query(Some(&*params));
    url.set_fragment(None);
    Ok(url.into())
}
```

# Appendix B - on the use of "URL"

The term "URL" refers to the concept of URL as defined in the [WHATWG URL spec].

It should not be confused with "URI". This is a deliberate choice because
browsers follow the WHATWG URL spec for parsing e.g. the `href` attribute in an
HTML `<a>` element, and this spec needs to be consistent with what browsers do.

[WHATWG URL spec]: https://url.spec.whatwg.org/
