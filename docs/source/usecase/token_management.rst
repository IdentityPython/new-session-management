================
Token Management
================

Token management starts when the authorization (or consent management)
module constructs a Grant instance. But first we need to start the
necessary services.

We use the following configuration:

.. code-block:: python


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
                    {"type": "oct", "bytes": "24", "use": ["enc"], "kid": "code"},
                    {"type": "oct", "bytes": "24", "use": ["enc"], "kid": "refresh"}
                ],
            },
            "code": {
                "kwargs": {"lifetime": 600}
            },
            "token": {
                "class": "oidcendpoint.token.jwt_token.JWTToken",
                "kwargs": {
                    "lifetime": 3600,
                    "base_claims": {"eduperson_scoped_affiliation": None},
                    "aud": ["https://example.org/appl"],
                },
            },
            "refresh": {
                "kwargs": {
                    "lifetime": 86400,
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

Based in that we initiate an Endpoint Context:

.. code-block:: python

    from oidcendpoint.endpoint_context import EndpointContext

    endpoint_context = EndpointContext(CONF)
    endpoint_context.cdb = {
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

And sets some shortcuts:

.. code-block:: python

    claims_interface = endpoint_context.claims_interface
    session_manager = endpoint_context.session_manager
    authn_endpoint = endpoint_context.endpoint["authorization"]
    introspection_endpoint = endpoint_context.endpoint["introspection"]

In the example we will use the following values::

    user_id = "diana"
    client_id = "client_2"
    resource_server = "https://rs.example.org/api"

Now it all starts with an authentication request. Which first leads
to the creation of a session

.. code-block:: python

    from oidcmsg.oidc import AuthorizationRequest
    from oidcendpoint.authn_event import create_authn_event

    AUTH_REQ = AuthorizationRequest(
        client_id="client_2",
        redirect_uri="https://example.com/cb",
        scope=["openid", "email", "phone"],
        state="STATE",
        response_type="code",
    )

    authn_event = create_authn_event(user_id)
    session_manager.create_session(authn_event, AUTH_REQ, user_id,
                                   client_id=client_id, sub_type="public")


and later after the user has had her say the creation of a grant. In
this example we just assume that the user is OK with the proposed release
of attributes:

.. code-block:: python

    from oidcendpoint.session.grant import Grant

    grant = Grant(scope=AUTH_REQ['scope'], resources=[client_id, resource_server])
    grant.claims = claims_interface.get_claims_all_usage(client_id, user_id, AUTH_REQ["scope"])

The grant is assigned to a session (user_id + client_id) this is reflected
in the path used in the command.
This command will store the grant in the session database.

.. code-block:: python

    session_manager.set([user_id, client_id, grant.id], grant)

and we can now create the session_key

.. code-block:: python

    from oidcendpoint.session import session_key

    session_id = session_key(user_id, client_id, grant.id)


Since the response type was code we need to mint a code.

.. code-block:: python

    from oidcmsg.time_util import time_sans_frac

    c_handler = session_manager.token_handler["code"]
    code = grant.mint_token(
        'authorization_code',
        value=c_handler(session_id),
        expires_at=time_sans_frac() + c_handler.lifetime
    )

The session_id will be included in the value. Which means we can
always find which session a token/code belongs to by looking into the
code.

Later we get an access token request


.. code-block:: python

    from oidcmsg.oidc import AccessTokenRequest
    TOKEN_REQ = AccessTokenRequest(
        client_id="client_2",
        redirect_uri="https://example.com/cb",
        state="STATE",
        grant_type="authorization_code",
        client_secret="hemligt",
        code=code.value
    )


The token endpoint in the OP verifies the request and then goes through
these steps (assuming that *code* now is what is referred to as *code.value*
above):

.. code-block:: python

    session_info = session_manager.get_session_info_by_token(code)
    authorization_code = session_manager.find_token(session_info["session_id"],
                                                    code)
    grant = session_info["grant"]

authorization_code is an AuthorizationCode instance fetched from the session
database. Minting an access token and an refresh token is now performed

.. code-block:: python

    at_handler = session_manager.token_handler["access_token"]
    access_token = grant.mint_token(
        'access_token',
        value=at_handler(session_info["session_id"]),
        expires_at=time_sans_frac() + at_handler.lifetime,
        based_on=authorization_code
    )

    rt_handler = session_manager.token_handler["refresh_token"]
    refresh_token = grant.mint_token(
        'refresh_token',
        value=rt_handler(session_info["session_id"]),
        expires_at=time_sans_frac() + rt_handler.lifetime,
        based_on=authorization_code
    )

and lastly but not least we need to mark the authorization code as used

.. code-block:: python

    authorization_code.register_usage()


Some time later a resource server may want to do introspection on
the access token os it sends the following request to the introspection
endpoint:

.. code-block:: python

    from oidcmsg.oauth2 import TokenIntrospectionRequest

    INTROSPECTION_REQUEST = TokenIntrospectionRequest(
        token=access_token.value,
        client_id="client_2",
        client_secret=introspection_endpoint.endpoint_context.cdb["client_2"]["client_secret"]
    )

The introspection endpoint handles this by doing

.. code-block:: python

    _req = introspection_endpoint.parse_request(INTROSPECTION_REQUEST)
    _resp = introspection_endpoint.process_request(_req)
    msg_info = introspection_endpoint.do_response(request=_req, **_resp)

Inside process_request the by now familiar sequence is applied

.. code-block:: python

    _session_info = session_manager.get_session_info_by_token(token)
    _token = session_manager.find_token(_session_info["session_id"], token)

If we print the introspection response we would see something like this::

    {
        "active": true,
        "scope": "openid email phone",
        "client_id": "client_2",
        "token_type": "access_token",
        "exp": 1607245745,
        "iat": 1607242145,
        "sub": "dcd5a00b58074dfff4e268d58fd4e066aea7a1efa09b407aaa39c110de518938",
        "iss": "https://example.com/",
        "aud": ["client_2", "https://example.com/api"]
    }

A bit later the access token has timed out and a new one needs to be minted

.. code-block:: python

    from oidcmsg.oidc import RefreshAccessTokenRequest

    REFRESH_TOKEN_REQ = RefreshAccessTokenRequest(
        grant_type="refresh_token",
        refresh_token=refresh_token.value,
        client_id="client_2",
        client_secret="hemligt"
    )

The request is parsed and if it's OK the op does (and you've seen this before),
refresh_token taken from the request.

.. code-block:: python

    session_info = session_manager.get_session_info_by_token(refresh_token)
    refresh_token = session_manager.find_token(session_info["session_id"],
                                               refresh_token)
    grant = session_info["grant"]

Now at this point someone decides that the new access token will not have the
same scope as the first one (no phone). So we take care of that when minting.

.. code-block:: python

    at_handler = session_manager.token_handler["access_token"]
    access_token2 = grant.mint_token(
        'access_token',
        value=at_handler(session_info["session_id"]),
        expires_at=time_sans_frac() + at_handler.lifetime,
        based_on=refresh_token,
        scope=["openid", "email"]
    )

Later still: the resource server may want to introspect the access token

.. code-block:: python

    introspection_request = TokenIntrospectionRequest(
        token=access_token2.value,
        client_id="client_2",
        client_secret=introspection_endpoint.endpoint_context.cdb["client_2"]["client_secret"]
    )

    _req = introspection_endpoint.parse_request(introspection_request)
    _resp = introspection_endpoint.process_request(_req)
    msg_info = introspection_endpoint.do_response(request=_req, **_resp)


The response will look something like this::

    {
        "active": true,
        "scope": "openid email",
        "client_id": "client_2",
        "token_type": "access_token",
        "exp": 1607246527,
        "iat": 1607242927,
        "sub": "3ea3c43cedf565696f4b97009da72069bd47013972d01dd2798c52d58dbb0ed6",
        "iss": "https://example.com/",
        "aud": ["client_2", "https://rs.example.org/api"]}

As you can see the scope of the access token has changed. The rest is the same
apart from *exp* and *iat* which of course is different.
