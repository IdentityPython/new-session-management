====================
Claims Interface API
====================

The claims interface is there to gather the set of claims that can be
returned for a specific use case. There are three use cases defined:

.. _usecases:

userinfo
    This what is returned at the userinfo endpoint

id_token
    Claims that can be returned in an ID Token

introspection
    Combines 2 use cases. The first is when the access token is in fact a
    JSON Web Token that can contains user claims. The second is what's returned
    from the introspection endpoint.

These are the methods provided by the interface.

    - authorization_request_claims_
    - get_claims_
    - get_user_claims_

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
    The use case. One of the usecases_ .


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
    Where the claims are going to be used. One of the usecases_ .


get_user_claims
---------------
.. _get_user_claims:

Use a set of permitted claims as a filter to figure out which claims
of the complete set of user claims to return.

.. _`OIDC Core`: http://openid.net/specs/openid-connect-core-1_0.html