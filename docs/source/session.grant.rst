
======
Grants
======

Grants are created by an authorization subsystem. If the grant is
created in connection with an user authentication the authorization system
might normally ask the user for usage consent and then base the construction
of the grant on that consent.

If an authorization server can act as a Security Token Service (STS) as
defined by https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-16
the user might set up one or more exchange. More about that when we get
to ExchangeGrant_ .

.. _Grant:
.. code-block:: python

    parameters = ["scope", "claim", "resources", "authorization_details",
                  "issued_token", "usage_rules", "revoked", "issued_at",
                  "expires_at"]
    type = "grant"

    def __init__(self,
                 scope: Optional[list] = None,
                 claims: Optional[dict] = None,
                 resources: Optional[list] = None,
                 authorization_details: Optional[dict] = None,
                 issued_token: Optional[list] = None,
                 usage_rules: Optional[dict] = None,
                 issued_at: int = 0,
                 expires_in: int = 0,
                 expires_at: int = 0,
                 revoked: bool = False,
                 token_map: Optional[dict] = None):

Grant is a subclass of :ref:`Item`

*scope*, *claims* and *resources* specifies restrictions that are to be applied
to tokens minted by this grant. *authorization_details* is a placeholder for
the time being. *issued_token* is a list of tokens minted by this grant.
*usage_rules* are templates to apply to the tokens minted by this grant.
*token_map* is a map between token types and classes to be used when
initiating such tokens.

-----

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



------

.. code-block:: python

    def to_json():

Converts the information in the instance into a string representation of a
JSON object. This string is what is expected to be stored in the database.

------

.. code-block:: python

    def from_json(json_str)

Sets attributes in the instance to values that are stored as a the
string representation of a JSON object. This method is used to fill a
instance with information stored about it in the database.
Code example:

.. code-block:: python

    from oidcendpoint.session.grant import Grant
    grant = Grant(scope=["openid", "foo", "bar"],
                  claims={"userinfo": {"given_name": None}},
                  resources=["https://api.example.com"])

    _json_str = grant.to_json()

    _new_code = Grant().from_json(_json_str)

    for attr in Grant.parameters:
        assert getattr(code, attr) == getattr(_new_code, attr)


------

.. code-block:: python

    def mint_token(
            token_type: str,
            value: str,
            based_on: Optional[Token] = None,
            usage_rules: Optional[dict] = None,
            **kwargs
            ) -> Optional[Token]:

Can be used to create new tokens. Based on another token or just on the
grant itself.
*token_type* is the type of token. By default the set::

        - authorization_code,
        - access_token and
        - refresh_token

is recognized. *value* is the value of the token. This is what is sent around
in OIDC protocol exchanges. *based_on*, a token the new token is a child of.
*kwargs* are extra keyword arguments that are used as parameter for the
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


------

.. code-block:: python

    def get_token(value: str) -> Optional[Token]

Among all the tokens that has been minted using this specific grant, find
the one that matches the *value* given. Usage example:

.. code-block:: Python

    from oidcendpoint.session.grant import Grant
    grant = Grant()
    code = grant.mint_token("authorization_code", value="ABCD")

    code_copy = grant.get_token(code.value)
    assert code_copy.id == code.id

------

.. _grant_revoke_token:

.. code-block:: python

    def revoke_token(
             value: Optional[str] = "",
             based_on: Optional[str] = "",
             recursive: bool = True):

Mark the token as revoked. *value* is the token value. *based_on* a reference
to the item this token is based on. *recursive* states whether all
descendants of a token that matches the search criteria will be also
marked as revoked.

.. code-block:: Python

    from oidcendpoint.session.grant import Grant
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

------

.. _grant_get_spec:
.. code-block:: python

    def get_spec(token: Token) -> Optional[dict]:

Claims, scope and resources can be specified for all tokens bound to a
grant by setting those attributes off the grant instance. It is also possible
to set specific values for specific tokens by setting those attributes in the
token. This method will return the token specific values if they exist otherwise
it will return the grant values for claims, scope and resources.

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



Exchange Grant
--------------
.. _ExchangeGrant:

Subclass of Grant_ .
