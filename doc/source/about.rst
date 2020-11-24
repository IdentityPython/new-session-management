.. _about_session_management:

========================
About session management
========================

The OIDC Session Management draft defines session to be:

    Continuous period of time during which an End-User accesses a Relying
    Party relying on the Authentication of the End-User performed by the
    OpenID Provider.

Note that we are dealing with a Single Sign On (SSO) context here.
If for some reason the OP does not want to support SSO then the
session management has to be done a bit differently. In that case each
session (user_id,client_id) would have its own authentication even. Not one
shared between the sessions.

    - `Design criteria`_
    - `Database layout`_

Design criteria
+++++++++++++++
.. _`Design criteria`:

So a session is defined by a user and a Relying Party. If one adds to that
that a user can have several sessions active at the same time each one against
a unique Relying Party we have the bases for session management.

Furthermore the user may well decide on different rules for different
relying parties for releasing user
attributes, where and how issued access tokens could be used and whether
refresh tokens should be issued or not.

We also need to keep track on which tokens where used to mint new tokens
such that we can easily revoked a suite of tokens all with a common ancestor.

Database layout
+++++++++++++++
.. _`Database layout`:

The database is organized in 3 levels. The top one being the users.
Below that the Relying Parties and at the bottom what is called grants.

Grants organize authorization codes, access tokens and refresh tokens (and
possibly other types of tokens) in a comprehensive way. More about that below.

There may be many Relying Parties below a user and many grants below a
Relying Party.
