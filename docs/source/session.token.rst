=============
Session Token
=============

Item
----
.. _Item:

Base class for both Token_ and :ref:`Grant`

.. code-block:: python

    def __init__(
         usage_rules: Optional[dict] = None,
         issued_at: int = 0,
         expires_in: int = 0,
         expires_at: int = 0,
         not_before: int = 0,
         revoked: bool = False,
         used: int = 0
    )

*usage_rules* applies to items that are minted based on this item.
*issued_at*, *expires_in*, *expires_at* and *not_before* are time
stamps that governs when this item is active. If *expires_at* is set to
zero and *expires_in* has a value > 0 then *expires_at* is calculated
as now + *expires_in*. If both *expires_in* and *expires_at* is zero
that means the item will never expire.
*revoked* marks the item as revoked or not revoked.
*used* are there to keep a record of how often the item has been used.

------

.. code-block:: python

    def max_usage_reached()

Returns **True** if the item has been used the maximum number of times
it is allowed to be used.

------

.. code-block:: python

    def is_active(now=0)

Returns **True** if the item can still be used.
This means that:

    - The time before which the item can not be used has passed,
    - The expiration time has not been reached,
    - The item has not been revoked and
    - The usage has not passed the max usage limit.

------

.. code-block:: python

    def revoke()

Sets the revoke attribute to **True**.
Code example:

.. code-block:: Python

    from oidcendpoint.session.token import Item
    token = Item()
    token.revoke()
    assert token.is_active() is False

Token
-----
.. _Token:

Token is a subclass of Item_. Token is the base class for different types
of tokens like:

    - AuthorizationCode_
    - AccessToken_
    - RefreshToken_

.. code-block:: python

    attributes = ["type", "issued_at", "not_before", "expires_at", "revoked",
                  "value", "usage_rules", "used", "based_on", "id", "scope",
                  "claims", "resources"]

    def __init__(
                 type: str = '',
                 value: str = '',
                 based_on: Optional[str] = None,
                 usage_rules: Optional[dict] = None,
                 issued_at: int = 0,
                 expires_in: int = 0,
                 expires_at: int = 0,
                 not_before: int = 0,
                 revoked: bool = False,
                 used: int = 0,
                 id: str = "",
                 scope: Optional[list] = None,
                 claims: Optional[dict] = None,
                 resources: Optional[list] = None,
                 ):

*type* is the type of token. This is used when retrieving the
object from storage such that the appropriate class can be initiated from the
information stored in the database.
*value* is what is actually sent around in the OIDC protocol exchange.
*based_on* points to which item that was used to initiate this item. Rules as
to which type of items that can be used to initiated what other types of
items are kept in *usage_rules* under the heading *supports_minting*.
*id* is a key that is used in the database. *scope* and *claims* sets
restrictions in the usage of the token. *resources* lists the servers for
which this token is valid.

------

.. code-block:: python

    def register_usage()

Expected to be used to register usage. Increments the used attribute.

------

.. code-block:: python

    def has_been_used()

Returns **True** if the Token has been used.
Code example:

.. code-block:: Python

    from oidcendpoint.session.token import Token
    token = Token(usage_rules={"max_usage": 2})

    token.register_usage()
    assert token.has_been_used() is True
    assert token.used == 1
    assert token.max_usage_reached() is False

    token.register_usage()
    assert token.max_usage_reached() is True
    assert token.used == 2

------

.. code-block:: python

    def to_json():

Converts the information in the instance into a string representation of a
JSON object. This string is what is expected to be stored in the database.

------

.. code-block:: python

    def from_json(json_str)

Sets attributes in the Token instance to values that are stored as a the
string representation of a JSON object. This method is used to fill a
token instance with information stored about it in the database.
Code example:

.. code-block:: python

    from oidcendpoint.session.token import Token
    token = Token(value="ABCD",
                  scope=["openid", "foo", "bar"],
                  claims={"userinfo": {"given_name": None}},
                  resources=["https://api.example.com"])

    _json_str = token.to_json()

    _new_code = Token().from_json(_json_str)

    for attr in Token.attributes:
        assert getattr(code, attr) == getattr(_new_code, attr)


------

.. code-block:: python

    def supports_minting(token_type)

Returns **True** if the token can support the initiation of a token of the
type *token_type*.

A token example::

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
        "id": "96d19bea275211eba43bacde48001122",
        "resources": ["https://example.com/api"]
    }


AuthorizationCode
-----------------
.. _AuthorizationCode:

Subclass of Token_. By default an AuthorizationCode instance can be used
to mint AccessToken_, RefreshToken_ and ID Tokens but not an AuthorizationCode_.
Also be default it can only be used ones. Minting an access_token, an
refresh_token and an ID token at the same time counts as one usage.

Example code:

.. code-block:: python

    from oidcendpoint.session.token import AuthorizationCode
    code = AuthorizationCode(value="ABCD")
    assert code.supports_minting('access_token')
    assert code.supports_minting('refresh_token')
    assert code.supports_minting("authorization_code") is False

AccessToken
-----------
.. _AccessToken:

Subclass of Token_

RefreshToken
------------
.. _RefreshToken:

Subclass of Token_

