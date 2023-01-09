# Fedi-To API

Fedi-To has two navigable endpoints. That is, these are endpoints the user is
required to navigate to, as by clicking a link.

## `GET /go`

### Parameters

- `h`, optional - the fallback handler to use
- `target` - the target URL

### Functional description

Attempts to send the user to a previously registered handler, if any.
Otherwise, attempts to send the user to the fallback handler, if present.
Otherwise, attempts to derive a fallback handler from the URL and send the user
to it.

### Examples

https://fedi-to.net/go?h=2&target=web%2Bganarchy:cc8ab1441fd398d23454c149c74bd5a2304eb566

https://fedi-to.net/go?target=web%2Bganarchy://ganarchy.github.io/cc8ab1441fd398d23454c149c74bd5a2304eb566

## `GET /register`

### Parameters

- `h` - the handler to register
- `protocol` - the protocol to handle

### Functional description

Allows registering a handler. Displays information about the handler so the
user can make a consensual decision.

### Examples

https://fedi-to.net/register?h=2&protocol=web%2Bganarchy
