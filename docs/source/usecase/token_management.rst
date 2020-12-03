================
Token Management
================

Token management starts when the authorization (or consent management)
module constructs a Grant instance.

There are a couple of things that goes into the Grant instance, these are
in no particular order:

claims
    The set of claims that the user is OK with releasing. Here the claims
    interface can help. More about that here :ref:`ClaimsInterface`.

scope
    This is really up to the authorization systems and is not a
    matter for the session/grant management system.

usage_rules
    Under what circumstances can a token be minted and how long are
    they valid.

expires_in
    How long the grant is valid. When as grant expires all tokens issued based
    on that grant will not automatically become inactive. When a grant expires
    no new tokens can be minted based on that grant.

