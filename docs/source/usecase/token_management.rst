================
Token Management
================

Token management starts when the authorization (or consent management)
module constructs a Grant instance.

One useful API in this instance is the ClaimsInterface. So let us
look at what it can do.

There are 4 cases where user information can be exported from an OP:

- userinfo endpoint
- ID Token
- introspection endpoint
- access token (if expressed as a JWT)

And there are 3 places where we through configuration can control which
claims are exported:

base_claims:
    This is claims that we want to export disregarding who gets it and
    ignoring the authentication request.

add_claim_by_scope
    Scopes are in some case short hand for sets of claims. There are a number
    of those defined in `OIDC Core`_.

enable_claims_per_client
    It is possible to add configuration information to specific clients. If so
    the claims are to be found in attributes by the name X_claims where X
    is one of userinfo, introspection, id_token or token.

Putting this together we get a matrix. As an example we use these values:

+--------------------------+-------------+---------------+-------------+----------+
| where                    | userinfo    | introspection | token       | id_token |
+==========================+=============+===============+=============+==========+
| base_claims              | email       |               | affiliation | email    |
|                          | affiliation |               |             |          |
+--------------------------+-------------+---------------+-------------+----------+
| add_claim_by_scope       | True        | False         | False       | True     |
+--------------------------+-------------+---------------+-------------+----------+
| enable_claims_per_client | True        | True          | False       | False    |
+--------------------------+-------------+---------------+-------------+----------+

This is then expressed in the following endpoint_context configuration:

.. code-block:: Python

    CONF = {
        "issuer": "https://example.com/",
        "password": "mycket hemlig zebra",
        "verify_ssl": False,
        "capabilities": {
            "response_types_supported": [
                ["code"],
            ],
            "token_endpoint_auth_methods_supported": [
                "private_key_jwt",
            ],
            "response_modes_supported": ["query"],
            "subject_types_supported": ["public", "pairwise", "ephemeral"],
            "grant_types_supported": [
                "authorization_code",
                "implicit",
                "urn:ietf:params:oauth:grant-type:jwt-bearer",
                "refresh_token",
            ],
            "claim_types_supported": ["normal", "aggregated", "distributed"],
            "claims_parameter_supported": True,
            "request_parameter_supported": True,
            "request_uri_parameter_supported": True,
        },
        "keys": {
            "uri_path": "jwks.json",
            "key_defs": [
                {"type": "RSA", "use": ["sig"]},
                {"type": "EC", "crv": "P-256", "use": ["sig"]},
            ]
        },
        "token_handler_args": {
            "jwks_def": {
                "private_path": "private/token_jwks.json",
                "read_only": False,
                "key_defs": [
                    {"type": "oct", "bytes": "24", "use": ["enc"], "kid": "code"}
                ],
            },
            "code": {"lifetime": 600},
            "token": {
                "class": "oidcendpoint.jwt_token.JWTToken",
                "kwargs": {
                    "lifetime": 3600,
                    "base_claims": {"eduperson_scoped_affiliation": None},
                    "aud": ["https://example.org/appl"],
                },
            },
            "refresh": {
                "class": "oidcendpoint.jwt_token.JWTToken",
                "kwargs": {
                    "lifetime": 3600,
                    "aud": ["https://example.org/appl"],
                }
            }
        },
        "endpoint": {
            "authorization": {
                "path": "{}/authorization",
                "class": "oidcendpoint.oidc.authorization.Authorization",
                "kwargs": {"client_authn_method": None},
            },
            "introspection": {
                "path": "{}/intro",
                "class": "oidcendpoint.oauth2.introspection.Introspection",
                "kwargs": {
                    "client_authn_method": ["client_secret_post"],
                    "enable_claims_per_client": True,
                },
            },
            "token": {
                "path": "token",
                "class": "oidcendpoint.oidc.token.Token",
                "kwargs": {
                    "client_authn_methods": [
                        "client_secret_post",
                        "client_secret_basic",
                        "client_secret_jwt",
                        "private_key_jwt",
                    ]
                },
            },
            "userinfo": {
                "path": "userinfo",
                "class": "oidcendpoint.oidc.userinfo.UserInfo",
                "kwargs": {
                    "claim_types_supported": [
                        "normal",
                        "aggregated",
                        "distributed",
                    ],
                    "base_claims": {"email": None, "eduperson_scoped_affiliation": None},
                    "client_authn_method": ["bearer_header"],
                    "add_claims_by_scope": True,
                    "enable_claims_per_client": True
                },
            },
        },
        "authentication": {
            "anon": {
                "acr": "urn:oasis:names:tc:SAML:2.0:ac:classes:InternetProtocolPassword",
                "class": "oidcendpoint.user_authn.user.NoAuthn",
                "kwargs": {"user": "diana"},
            }
        },
        "userinfo": {
            "class": "oidcendpoint.user_info.UserInfo",
            "kwargs": {"db": "users.json"}
        },
        "template_dir": "template",
        "id_token": {
            "class": "oidcendpoint.id_token.IDToken",
            "kwargs": {
                "base_claims": {"email": None},
                "add_claims_by_scope": True
            }
        },
    }

Using this configuration we can now initiate an EndpointContext and add some
client configuration.

.. code-block:: Python

    endpoint_context = EndpointContext(CONF)
    endpoint_context.cdb = {
        "client_1": {
            "client_secret": "hemligt",
            "client_id": "client_1",
            "redirect_uris": [("https://client1.example.com/cb", None)],
            "client_salt": "salted",
            "token_endpoint_auth_method": "client_secret_post",
            "response_types": ["code"],
        },
        "client_2": {
            "client_id": "client_2",
            "client_secret": "hemligare",
            "redirect_uris": [("https://client2.example.org/cb", None)],
            "client_salt": "saltare",
            "token_endpoint_auth_method": "client_secret_post",
            "response_types": ["code"],
            "userinfo_claims": {"phone_number": None, "name": None},
            "introspection_claims": {"phone_number": None, "name": None}
        },
    }

    claims_interface = endpoint_context.claims_interface
    authn_endpoint = endpoint_context.endpoint["authorization"]

We simulate 2 sessions by processing 2 authentication requests from 2
different clients:

.. code-block:: Python

    # An authentication request from client_1
    AUTHN_REQ_1 = AuthorizationRequest(
        state="state1",
        response_type="code",
        redirect_uri="https://client1.example.com/cb",
        scope=["openid"],
        client_id="client_1",
    )

    _pr_resp = authn_endpoint.parse_request(AUTHN_REQ_1.to_dict())
    _resp = authn_endpoint.process_request(_pr_resp)
    _code2 = _resp["response_args"]["code"]

    # An authentication request from client_2
    AUTHN_REQ_2 = AuthorizationRequest(
        state="state2",
        response_type="code",
        redirect_uri="https://client2.example.org/cb",
        scope=["openid", "email", "address"],
        client_id="client_2",
    )

    _pr_resp = authn_endpoint.parse_request(AUTHN_REQ_2.to_dict())
    _resp = authn_endpoint.process_request(_pr_resp)

Now for the fun part. The method we want to use if **get_claims**.
It takes 4 arguments:

- client_id,
- user_id,
- scope and
- usage

If we want the claims to return to client_1 over the userinfo endpoint
interface we do:

.. code-block:: Python

    claims_interface.get_claims('client_1', "diana", AUTHN_REQ_1["scope"],
                                "userinfo")

Given the configuration above the result of that command will be dictionary
of the form::

    {'email': None, 'eduperson_scoped_affiliation': None, 'sub': None}

Running the same command but for both client sessions and all 4 interfaces
we get the following matrix with just the claims names.

+----------+----------------+---------------+----------------+-------+
| client   | userinfo       | introspection | id_token       | token |
+==========+================+===============+================+=======+
| client_1 | email          |               | email          |       |
|          | affiliation    |               | sub            |       |
|          | sub            |               |                |       |
+----------+----------------+---------------+----------------+-------+
| client_2 | email          | phone_number  | email          |       |
|          | affiliation    | name          | sub            |       |
|          | sub            |               | email_verified |       |
|          | phone_number   |               | address        |       |
|          | name           |               |                |       |
|          | email_verified |               |                |       |
|          | address        |               |                |       |
+----------+----------------+---------------+----------------+-------+

Note: I have abbreviated 'eduperson_scoped_affiliation' as affiliation

I leave it as an exercise for the read to verify the correctness of the
data.

Now to find the exact user information to return you can use the ClaimsInterface
method **get_user_claims**.

.. code-block:: Python

    _userinfo_restriction = claims_interface.get_claims('client_1',
                                                        "diana",
                                                        AUTHN_REQ_1["scope"],
                                                        "userinfo")

    res = self.claims_interface.get_user_claims("diana", _userinfo_restriction)

This would give you the exact user info to return over the interface in question.

Now to find out what to display to the user's consent page you would run:

.. code-block:: Python

    _claims = claims_interface.get_claims_all_usage('client_1',
                                                    "diana",
                                                    AUTHN_REQ_1["scope"])

    ava = self.claims_interface.get_user_claims("diana", _claims)


ava would then contain all the claims the OP can imaging returning to a
client and their values.

.. _`OIDC Core`: http://openid.net/specs/openid-connect-core-1_0.html