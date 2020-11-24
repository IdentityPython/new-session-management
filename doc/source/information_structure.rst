.. _`The information structure`:

=========================
The information structure
=========================

As stated above there are 3 layers: user session information, client session
information and grants. But first the keys to the information.

    - `Session key`_
    - `User session information`_
    - `Client session information`_
    - `Grant information`_
    - `Token`_

Session key
+++++++++++
.. _`Session key`:

A key to the session information is based on a list. The first item being the
user identifier, the second the client identifier and the third the grant
identifier.
If you only want the user session information then the key is a list with one
item the user id. If you want the client session information the key is a
list with 2 items (user_id, client_id). And lastly if you want a grant then
the key is a list with 3 elements (user_id, client_id, grant_id).

A *session identifier* is constructed using the **session_key** function.
It takes as input the 3 elements list.::

    session_id = session_key(user_id, client_id, grant_id)


Using the function **unpack_session_key** you can get the elements from a
session_id.::

    user_id, client_id, grant_id = unpack_session_id(session_id)


User session information
++++++++++++++++++++++++
.. _`User session information`:

Houses the authentication event information which is the same for all session
connected to a user.
Here we also have a list of all the clients that this user has a session with.
Expressed as a dictionary this can look like this::

    {
        'authentication_event': {
            'uid': 'diana',
            'authn_info': "urn:oasis:names:tc:SAML:2.0:ac:classes:InternetProtocolPassword",
            'authn_time': 1605515787,
            'valid_until': 1605519387
        },
        'subordinate': ['client_1']
    }


Client session information
++++++++++++++++++++++++++
.. _`Client session information`:

The client specific information of the session information.
Presently only the authorization request and the subject identifier (sub).
The subordinates to this set of information are the grants::

    {
        'authorization_request':{
            'client_id': 'client_1',
            'redirect_uri': 'https://example.com/cb',
            'scope': ['openid', 'research_and_scholarship'],
            'state': 'STATE',
            'response_type': ['code']
        },
        'sub': '117afe8d7bb0ace8e7fb2706034ab2d3fbf17f0fd4c949aa9c23aedd051cc9e3',
        'subordinate': ['e996c61227e711eba173acde48001122'],
        'revoked': False
    }

Grant information
+++++++++++++++++
.. _`Grant information`:

Grants are created by an authorization subsystem in an OP. If the grant is
created in connection with an user authentication the authorization system
might normally ask the user for usage consent and then base the construction
of the grant on that consent.

If an authorization server can act as a Security Token Service (STS) as
defined by https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-16
then no user is involved. In the context of session management the STS is
equivalent to a user.

Grant information contains information about user consent and issued tokens.::

    {
        "type": "grant",
        "scope": ["openid", "research_and_scholarship"],
        "authorization_details": null,
        "claims": {
            "userinfo": {
                "sub": null,
                "name": null,
                "given_name": null,
                "family_name": null,
                "email": null,
                "email_verified": null,
                "eduperson_scoped_affiliation": null
            }
        },
        "resources": ["client_1"],
        "issued_at": 1605452123,
        "not_before": 0,
        "expires_at": 0,
        "revoked": false,
        "issued_token": [
            {
                "type": "authorization_code",
                "issued_at": 1605452123,
                "not_before": 0,
                "expires_at": 1605452423,
                "revoked": false,
                "value": "Z0FBQUFBQmZzVUZieDFWZy1fbjE2ckxvZkFTVC1ZTHJIVlk0Z09tOVk1M0RsOVNDbkdfLTIxTUhILWs4T29kM1lmV015UEN1UGxrWkxLTkVXOEg1WVJLNjh3MGlhMVdSRWhYcUY4cGdBQkJEbzJUWUQ3UGxTUWlJVDNFUHFlb29PWUFKcjNXeHdRM1hDYzRIZnFrYjhVZnIyTFhvZ2Y0NUhjR1VBdzE0STVEWmJ3WkttTk1OYXQtTHNtdHJwYk1nWnl3MUJqSkdWZGFtdVNfY21VNXQxY3VzalpIczBWbGFueVk0TVZ2N2d2d0hVWTF4WG56TDJ6bz0=",
                "usage_rules": {
                    "expires_in": 300,
                    "supports_minting": [
                        "access_token",
                        "refresh_token",
                        "id_token"
                    ],
                    "max_usage": 1
                    },
                "used": 0,
                "based_on": null,
                "id": "96d19bea275211eba43bacde48001122"
           },
           {
                "type": "access_token",
                "issued_at": 1605452123,
                "not_before": 0,
                "expires_at": 1605452723,
                "revoked": false,
                "value": "Z0FBQUFBQmZzVUZiaWVRbi1IS2k0VW4wVDY1ZmJHeEVCR1hVODBaQXR6MWkzelNBRFpOS2tRM3p4WWY5Y1J6dk5IWWpnelRETGVpSG52b0d4RGhjOWphdWp4eW5xZEJwQzliaS16cXFCcmRFbVJqUldsR1Z3SHdTVVlWbkpHak54TmJaSTV2T3NEQ0Y1WFkxQkFyamZHbmd4V0RHQ3k1MVczYlYwakEyM010SGoyZk9tUVVxbWdYUzBvMmRRNVlZMUhRSnM4WFd2QzRkVmtWNVJ1aVdJSXQyWnpVTlRiZnMtcVhKTklGdzBzdDJ3RkRnc1A1UEw2Yz0=",
                "usage_rules": {
                    "expires_in": 600,
                },
                "used": 0,
                "based_on": "Z0FBQUFBQmZzVUZieDFWZy1fbjE2ckxvZkFTVC1ZTHJIVlk0Z09tOVk1M0RsOVNDbkdfLTIxTUhILWs4T29kM1lmV015UEN1UGxrWkxLTkVXOEg1WVJLNjh3MGlhMVdSRWhYcUY4cGdBQkJEbzJUWUQ3UGxTUWlJVDNFUHFlb29PWUFKcjNXeHdRM1hDYzRIZnFrYjhVZnIyTFhvZ2Y0NUhjR1VBdzE0STVEWmJ3WkttTk1OYXQtTHNtdHJwYk1nWnl3MUJqSkdWZGFtdVNfY21VNXQxY3VzalpIczBWbGFueVk0TVZ2N2d2d0hVWTF4WG56TDJ6bz0=",
                "id": "96d1c840275211eba43bacde48001122"
           }
        ],
        "id": "96d16d3c275211eba43bacde48001122"
    }

The parameters are described below

scope
:::::

This is the scope that was chosen for this grant. Either by the user or by
some rules that the Authorization Server runs by.

authorization_details
:::::::::::::::::::::

Presently a place hold. But this is expected to be information on how the
authorization was performed. What input was used and so on.

claims
::::::

The set of claims that should be returned in different circumstances. The
syntax that is defined in
https://openid.net/specs/openid-connect-core-1_0.html#ClaimsParameter
is used. With one addition, beside *userinfo* and *id_token* we have added
*introspection*.

resources
:::::::::

This are the resource servers and other entities that should be accepted
as users of issued access tokens.

issued_at
:::::::::

When the grant was created. Its value is a JSON number representing the number
of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.

not_before
::::::::::
If the usage of the grant should be delay, this is when it can start being used.
Its value is a JSON number representing the number
of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.

expires_at
::::::::::
When the grant expires.
Its value is a JSON number representing the number
of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.

revoked
:::::::
If the grant has been revoked.

issued_token
::::::::::::
Tokens that has been issued based on this grant. There is no limitation
as to which tokens can be issued. Though presently we only have:

- authorization_code,
- access_token and
- refresh_token

id
::
The grant identifier.

Token
+++++
.. _`Token`:

As mention above there are presently only 3 token types that are defined:

- authorization_code,
- access_token and
- refresh_token

A token is described as follows::

    {
        "type": "authorization_code",
        "issued_at": 1605452123,
        "not_before": 0,
        "expires_at": 1605452423,
        "revoked": false,
        "value": "Z0FBQUFBQmZzVUZieDFWZy1fbjE2ckxvZkFTVC1ZTHJIVlk0Z09tOVk1M0RsOVNDbkdfLTIxTUhILWs4T29kM1lmV015UEN1UGxrWkxLTkVXOEg1WVJLNjh3MGlhMVdSRWhYcUY4cGdBQkJEbzJUWUQ3UGxTUWlJVDNFUHFlb29PWUFKcjNXeHdRM1hDYzRIZnFrYjhVZnIyTFhvZ2Y0NUhjR1VBdzE0STVEWmJ3WkttTk1OYXQtTHNtdHJwYk1nWnl3MUJqSkdWZGFtdVNfY21VNXQxY3VzalpIczBWbGFueVk0TVZ2N2d2d0hVWTF4WG56TDJ6bz0=",
        "usage_rules": {
            "expires_in": 300,
            "supports_minting": [
                "access_token",
                "refresh_token",
                "id_token"
            ],
            "max_usage": 1
            },
        "used": 0,
        "based_on": null,
        "id": "96d19bea275211eba43bacde48001122"
    }


type
::::
The type of token.

issued_at
:::::::::
When the token was created. Its value is a JSON number representing the number
of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.

not_before
::::::::::
If the start of the usage of the token is to be delay, this is until when.
Its value is a JSON number representing the number
of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.

expires_at
::::::::::
When the token expires.
Its value is a JSON number representing the number
of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.

revoked
:::::::
If the token has been revoked.

value
:::::
This is the value that appears in OIDC protocol exchanges.

usage_rules
:::::::::::
Rules as to how this token can be used:

expires_in
    Used to calculate expires_at

supports_minting
    The tokens types that can be minted based on this token. Typically a code
    can be used to mint ID tokens and access and refresh tokens.

max_usage
    How many times this token can be used (being used is presently defined as
    used to mint other tokens). An authorization_code token can according to
    the OIDC standard only be used once but then to, in the same session,
    mint more then one token.

used
::::
How many times the token has been used

based_on
::::::::
Reference to the token that was used to mint this token. Might be empty if the
token was minted based on the grant it belongs to.

id
::
Token identifier
