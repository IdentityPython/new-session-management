=========
Grant API
=========

    - max_usage_reached_
    - is_active_
    - revoke_
    - update_
    - replace_
    - mint_token_
    - get_token_
    _ revoke_token_

max_usage_reached
-----------------
.. _max_usage_reached:

If there is an upper limit as to how many times a grant can be used to mint
new tokens this method will tell you if that limit has been reached.

is_active
---------
.. _is_active:

There might be several reasons why a grant can not be used to mint new tokens.
It may have expires, been revoked or have reached its max usage limit. This
method will tell you if any of those limits has been reached.

revoke
------
.. _revoke:

Will revoke a grant. Does not necessarily mean that all the tokens that has
been minted by this grant also will be revoked.

update
------
.. _update:

Will update information in the grant. This can only be used for the parameters:

    - authorization_details
    - claims
    - resources
    - scope

authorization_details and claims are dictionaries while resources and scope are
lists.

Thus if you want to update the claims specification you can do like this:

.. code-block:: python
    :linenos:

    from oidcendpoint.grant import Grant

    grant = Grant()
    user_info_claims = {"given_name": None, "email": None}
    grant.update(claims={"userinfo": user_info_claims})

    id_token_claims = {"email": None}
    grant.update(claims={"id_token": id_token_claims})

    introspection_claims = {"affiliation": None}
    grant.update(claims={"introspection": introspection_claims})

    assert set(grant.claims.keys()) == {"userinfo", "id_token", "introspection"}

If it is about resources it will be much the same:

.. code-block:: python
    :linenos:

    from oidcendpoint.grant import Grant

    grant = Grant()
    grant.update(resources=["https://api.example.com"])
    assert grant.resources == ["https://api.example.com"]

    grant.update(resources=["https://api.example.com",
                            "https://api.example.org"])

    assert set(grant.resources) == {"https://api.example.com",
                                    "https://api.example.org"}


replace
-------
.. _replace:

Replaces whatever value there is on the parameter with something new.
As with update this can used on the parameters:

    - authorization_details
    - claims
    - resources
    - scope

Code example:

.. code-block:: python
    :linenos:

    from oidcendpoint.grant import Grant

    grant = Grant()
    grant.update(resources= ["https://api.example.com"])
    print(grant.resources == ["https://api.example.com"])

    grant.replace(resources= ["https://api.example.org"])

    print(set(grant.resources) == {"https://api.example.org"})

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

.. code-block::

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




