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

The fallback protocol handler can be derived entirely from the URL, without the
need for any web requests. Using the Rust `url` crate, version `2.3.1`, the
fallback protocol handler is computed like so:

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
    let mut params = "target=".to_owned();
    params.extend(utf8_percent_encode(&*target, COMPONENT));
    url.set_query(Some(&*params));
    url.set_fragment(None);
    Ok(url.into())
}
```
