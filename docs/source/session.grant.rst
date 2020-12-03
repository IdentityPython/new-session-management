=====
Grant
=====

Grants are created by an authorization subsystem in an OP. If the grant is
created in connection with an user authentication the authorization system
might normally ask the user for usage consent and then base the construction
of the grant on that consent.

If an authorization server can act as a Security Token Service (STS) as
defined by https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-16
then no user is involved. In the context of session management the STS is
equivalent to a user.

Grant information
+++++++++++++++++
.. _`Grant information`:

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

The base class is Grant with the methods:

is_active
---------
.. _grant_is_active:

There might be several reasons why a grant can not be used to mint new tokens.
It may have expires, been revoked or have reached its max usage limit. This
method will tell you if any of those limits has been reached.

max_usage_reached
-----------------
.. _grant_max_usage_reached:

If there is an upper limit as to how many times a grant can be used to mint
new tokens this method will tell you if that limit has been reached.

revoke
------
.. _grant_revoke:

Will revoke a grant. Does not necessarily mean that all the tokens that has
been minted by this grant also will be revoked.

mint_token
----------
.. _grant_mint_token:

Can be used to create new tokens. Based on another token or just on the
grant itself. Method parameters are:

token_type
    The type of token. By default the set::

        - authorization_code,
        - access_token and
        - refresh_token

    is recognized.
value
    The value of the token. This what is sent around in OIDC protocol
    exchanges.
based_on
    A token the new token is a child of.
kwargs
    Extra keyword arguments that are used as parameter used in the
    token initialisation.

Code example:

.. code-block:: python

    from oidcendpoint.grant import Grant
    grant = Grant()
    code = grant.mint_token("authorization_code", value="ABCD")
    access_token = grant.mint_token("access_token",
                                    value="1234",
                                    based_on=code,
                                    scope=["openid", "foo", "bar"])

    assert access_token.scope == ["openid", "foo", "bar"]


get_token
---------
.. _grant_get_token:

Among all the tokens that has been minted using this specific grant, find
the one that matches the value given.
Takes only one argument: the value.

.. code-block:: Python

    from oidcendpoint.grant import Grant
    grant = Grant()
    code = grant.mint_token("authorization_code", value="ABCD")

    _code = grant.get_token(code.value)
    assert _code.id == code.id

revoke_token
------------
.. _grant_revoke_token:

Mark the token as revoked.
Takes three arguments:

value
    The token value

based_on:
    A token index

recursive:
    A boolean. If true it means that all descendants of a token
    that matches the search criteria will be also marked as revoked.

.. code-block:: Python

    from oidcendpoint.grant import Grant
    grant = Grant()
    code = grant.mint_token("authorization_code", value="ABCD")
    access_token = grant.mint_token("access_token", value="1234", based_on=code)

    grant.revoke_token(based_on=code.value)

    assert code.is_active() is True
    assert access_token.is_active() is False

    access_token_2 = grant.mint_token("access_token",
                                      value="0987", based_on=code)

    grant.revoke_token(value=code.value, recursive=True)

    assert code.is_active() is False
    assert access_token_2.is_active() is False

get_spec
--------
.. _grant_get_spec:

Claims, scope and resources can be specified for all tokens bound to a
grant by setting those attributes off the grant instance. It is also possible
to set specific values for specific tokens by setting those attributes in the
token. This method will return the token specific values if they exist otherwise
it will return the grant values for claims, scpoe and resources.

.. code-block:: Python

    from oidcendpoint.grant import Grant
    grant = Grant(scope=["openid", "email", "address"],
                  claims={"userinfo": {"given_name": None, "email": None}},
                  resources=["https://api.example.com"]
                  )
    code = grant.mint_token("authorization_code", value="ABCD")
    access_token = grant.mint_token("access_token", value="1234", based_on=code,
                                    scope=["openid", "email", "eduperson"],
                                    claims={
                                        "userinfo": {
                                            "given_name": None,
                                            "eduperson_affiliation": None
                                        }
                                    })

    spec = grant.get_spec(access_token)
    assert set(spec.keys()) == {"scope", "claims", "resources"}
    assert spec["scope"] == ["openid", "email", "eduperson"]
    assert spec["claims"] == {
        "userinfo": {
            "given_name": None,
            "eduperson_affiliation": None
        }
    }
    assert spec["resources"] == ["https://api.example.com"]



