.. _session_claims:

======
Claims
======

The claims interface is there to gather the set of claims that can be
returned for a specific use case. There are three interfaces defined:

.. _interfaces:

userinfo
    This what is returned at the userinfo endpoint

id_token
    Claims that can be returned in an ID Token

introspection
    What is returned from the introspection endpoint.

token
    An access token in the form of a
    JSON Web Token that can contains user claims.

These are the methods provided by the interface.

authorization_request_claims
----------------------------
.. _authorization_request_claims:

According to section 5.5 of `OIDC Core`_ a client can ask for a specific
set of claims by using the claims parameter in the authentication request.
This method picks the claim set from the request.

Methods arguments:

user_id
    User identifier

client_id
    Client identifier

usage
    The use case. One of the interfaces_.


get_claims
----------
.. _get_claims:

Gathers information about which claims to return from a number of places.
This is based on endpoint configuration. The configuration parameters used are:

base_claims
    One of the parameters of the endpoint configuration is *base_claims*.
    This defines a basic set to return over that interface.

enable_claims_per_client
    Another endpoint configuration parameter. If set to True this enables the
    OP to have per client definitions of what to return.
    The claims specification is stored in the client configuration as values
    of the attribute <usage>_claims (userinfo_claims, id_token_claims,...).

add_claims_by_scope
    Some scopes defined in `OIDC Core`_ equates to sets of claims.
    This parameter governs whether scopes provided in the authentication
    request should be translated into claims or not

The parameters to the method is:

user_id
    User identifier

client_id
    Client identifier (client_id)

scopes
    Scopes from the authentication request

usage
    Where the claims are going to be used. One of the interfaces_.

Assume the following configuration of the userinfo endpoint::

    "userinfo": {
        "path": "userinfo",
        "class": userinfo.UserInfo,
        "kwargs": {
            "claim_types_supported": [
                "normal",
                "aggregated",
                "distributed",
            ],
            "client_authn_method": ["bearer_header"],
            "base_claims": {
                "eduperson_scoped_affiliation": None,
                "email": None,
            },
            "add_claims_by_scope": True,
            "enable_claims_per_client": True
        },
    },

and the following authentication request::

    AUTHN_REQ = AuthorizationRequest(
        response_type="code",
        client_id="client1",
        redirect_uri="http://example.com/authz",
        scope=["openid"],
        state="state000",
        claims={
            "userinfo": {
                "eduperson_scoped_affiliation": {"essential": True},
                "nickname": None,
                "email": {"essential": True},
                "email_verified": {"essential": True},
            }
        }
    )

.. code-block:: Python

    endpoint_context = EndpointContext(CONFIG)
    session_manager = endpoint_context.session_manager
    claims_interface = ClaimsInterface(endpoint_context)

    user_id = "diana"
    client_id = AUTHN_REQ["client_id]
    authn_event = create_authn_event(user_id)

    session_manager.create_session(authn_event, AUTHN_REQ, user_id, client_id=client_id)

    _userinfo_restriction = claims_interface.get_claims(client_id=_cid, user_id=_uid,
                                                        scopes=OIDR["scope"],
                                                        usage="userinfo")

    assert _userinfo_restriction == {'eduperson_scoped_affiliation': None,
                                     'email': None}

    res = claims_interface.get_user_claims("diana", _userinfo_restriction)

    assert res == {
        'eduperson_scoped_affiliation': ['staff@example.org'],
        "email": "diana@example.org",
    }

What get_claims does is first fetch the base claims from the endpoint
configuration. In this case that is::

    "base_claims": {
        "eduperson_scoped_affiliation": None,
        "email": None,
    },

Since *add_claims_by_scope* is defined as True get_claims will then
convert the scopes into sets of claims. In this case it adds *sub*.
Finally since *enable_claims_per_client* is set to True it will look in the
client configuration and find nothing. So the end result of the claims
gathering are the base claims plus *sub*. That is then matched against the
claims requests in the authentication request. What we are looking for here is
the intersection between what get_claims has so far with the requested claims.
The final result is that *eduperson_scoped_affiliation* and *email* are
matched against what is in the user database and *nickname* and
*email_verified* are ignored. *sub* is a special case since according to
Section 5.3.2 of `OIDC Core`_ ::

    The sub (subject) Claim MUST always be returned in the UserInfo Response.

Where you would use the result you get from *get_user_claims* is in the
consent interaction with the user.

get_user_claims
---------------
.. _get_user_claims:

Use a set of permitted claims as a filter to figure out which claims
of the complete set of user claims to return.

.. _`OIDC Core`: http://openid.net/specs/openid-connect-core-1_0.html