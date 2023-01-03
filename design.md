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
