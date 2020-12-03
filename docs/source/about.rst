.. _about_session_management:

========================
About session management
========================

The Session Management subsystem are designed to support a couple of
different services that we have in, or are in the process of adding to, the
system as it is:

Single Logout
    Described in a set of specifications:
        - `Session Management`_
        - `Back-Channel Logout`_
        - `RP-Initiated Logout`_
        - `Front-Channel Logout`_

Authorization management
    Information about which tokens can be issued and what they are
    supposed to be used for and by whom, must be managed somewhere.

Token exchange
    As described in `RFC 8693`_ - OAuth 2.0 Token Exchange

And some new functionality that we might want to support:

`OAuth 2.0 Rich Authorization Requests`_
    Allows clients to specify their fine-grained authorization
    requirements using the expressiveness of JSON data structures.

`FAPI Grant management`_
    Wants to extend OAuth to include explicit representation of grants in
    the OAuth protocol.


The OIDC Session Management draft defines session to be:

    Continuous period of time during which an End-User accesses a Relying
    Party relying on the Authentication of the End-User performed by the
    OpenID Provider.

Note that we are dealing with a Single Sign On (SSO) context here.
If for some reason the OP does not want to support SSO then the
session management has to be done a bit differently. In that case each
session (user_id,client_id) would have its own authentication even. Not one
shared between the sessions.

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

There may be many Relying Parties below an user and many grants below a
Relying Party.

Session key
+++++++++++
.. _`Session key`:

The key to the session information is based on a list. The first item being the
user identifier, the second the client identifier and the third the grant
identifier.
If you only want the user session information then the key is a list with one
item, the user id. If you want the client session information the key is a
list with 2 items (user_id, client_id). And lastly if you want a grant then
the key is a list with 3 elements (user_id, client_id, grant_id).

A *session identifier* is constructed using the **session_key** function.
It takes as input 3 elements.::

    session_id = session_key(user_id, client_id, grant_id)


Using the function **unpack_session_key** you can get the elements from a
session_id.::

    user_id, client_id, grant_id = unpack_session_id(session_id)


.. _`Session Management`: https://openid.net/specs/openid-connect-session-1_0.html
.. _`Back-Channel Logout`: https://openid.net/specs/openid-connect-backchannel-1_0.html
.. _`RP-Initiated Logout`: https://openid.net/specs/openid-connect-rpinitiated-1_0.html
.. _`Front-Channel Logout`: https://openid.net/specs/openid-connect-frontchannel-1_0.html
.. _`RFC 8693`: https://tools.ietf.org/html/rfc8693
.. _`OAuth 2.0 Rich Authorization Requests`: https://tools.ietf.org/html/draft-ietf-oauth-rar-03
.. _`FAPI Grant Management`: https://bitbucket.org/openid/fapi/src/master/Financial_API_Grant_Management.md
