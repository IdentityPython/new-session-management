=========
Grant API
=========

    - is_active_
    - max_usage_reached_
    - revoke_
    - mint_token_
    - get_token_
    - revoke_token_
    - get_spec_

is_active
---------
.. _is_active:

There might be several reasons why a grant can not be used to mint new tokens.
It may have expires, been revoked or have reached its max usage limit. This
method will tell you if any of those limits has been reached.

max_usage_reached
-----------------
.. _max_usage_reached:

If there is an upper limit as to how many times a grant can be used to mint
new tokens this method will tell you if that limit has been reached.

revoke
------
.. _revoke:

Will revoke a grant. Does not necessarily mean that all the tokens that has
been minted by this grant also will be revoked.

mint_token
----------
.. _mint_token:

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
    :linenos:

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
.. _get_token:

Among all the tokens that has been minted using this specific grant, find
the one that matches the value given.
Takes only one argument: the value.

.. code-block::

    from oidcendpoint.grant import Grant
    grant = Grant()
    code = grant.mint_token("authorization_code", value="ABCD")

    _code = grant.get_token(code.value)
    assert _code.id == code.id

revoke_token
------------
.. _revoke_token:

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
.. _get_spec:

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



