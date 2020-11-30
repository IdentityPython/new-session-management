======================================
Single Sign On/Single log out use case
======================================

We start with a configuration like this::

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
        "endpoint": {
            "authorization": {
                "path": "{}/authorization",
                "class": "oidcendpoint.oidc.authorization.Authorization",
                "kwargs": {"client_authn_method": None},
            },
            "session": {
                "path": "{}/end_session",
                "class": "oidcendpoint.oidc.session.Session",
                "kwargs": {
                    "post_logout_uri_path": "post_logout",
                    "signing_alg": "ES256",
                    "logout_verify_url": "https://example.com/verify_logout",
                    "client_authn_method": None,
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
    }


Given this configuration we can instantiate an endpoint context::

    from oidcendpoint.endpoint_context import EndpointContext
    endpoint_context = EndpointContext(CONF)

Add some client configuration::

    endpoint_context.cdb = {
        "client_1": {
            "client_secret": "hemligt",
            "client_id": "client_1",
            "redirect_uris": [("https://client1.example.com/cb", None)],
            "client_salt": "salted",
            "token_endpoint_auth_method": "client_secret_post",
            "response_types": ["code", "token", "code id_token", "id_token"],
            "post_logout_redirect_uris": [("https://client1.example.com/logout_cb", "")],
            "backchannel_logout_uri": "https://client1.example.com/bc_logout"
        },
        "client_2": {
            "client_id": "client_2",
            "client_secret": "hemligare",
            "redirect_uris": [("https://client2.example.org/cb", None)],
            "client_salt": "saltare",
            "token_endpoint_auth_method": "client_secret_post",
            "response_types": ["code", "token", "code id_token", "id_token"],
            "post_logout_redirect_uris": [("https://client2.example.org/logout_cb", "")],
            "frontchannel_logout_uri": "https://client2.example.org/fc_logout"
        },
    }

and pick out some links for later usage::

    session_manager = endpoint_context.session_manager
    authn_endpoint = endpoint_context.endpoint["authorization"]
    session_endpoint = endpoint_context.endpoint["session"]

Now, when the setup is done. We'll have the first client send an authentication
request:

.. code-block:: python

    from oidcmsg.oidc import AuthorizationRequest

    AUTHN_REQ_1 = AuthorizationRequest(
        state="state1",
        response_type="code",
        redirect_uri="https://client1.example.com/cb",
        scope=["openid"],
        client_id="client_1",
    )

    _pr_resp = authn_endpoint.parse_request(AUTHN_REQ_1.to_dict())
    _resp = authn_endpoint.process_request(_pr_resp)
    _code1 = _resp["response_args"]["code"]

The request parsing and processing went OK and the response contains a
*code* value.
Now we can look at the session information:

.. code-block:: python

    _session_info_1 = session_manager.get_session_info_by_token(_code1)

and *_session_info_1* will have values like

+---------------------+-----------------------------------------------------+
| key                 |  value                                              |
+=====================+=====================================================+
| session_id          | diana:client_1:f3fcaf84322511eba12cacde48001122     |
+---------------------+-----------------------------------------------------+
| client_id           | client_1                                            |
+---------------------+-----------------------------------------------------+
| user_id             | diana                                               |
+---------------------+-----------------------------------------------------+
| user_session_info   | :authentication_event:                              |
|                     |    :uid: "diana"                                    |
|                     |    :authn_info: "urn:oasis:names:tc:.."             |
|                     |    :authn_time: 1606642406                          |
|                     |    :valid_until: 1606646006                         |
|                     | :subordinate: ['client_1']                          |
|                     | :revoked': False                                    |
|                     | :type': 'UserSessionInfo'                           |
+---------------------+-----------------------------------------------------+
| client_session_info | :authorization_request:                             |
|                     |    <oidcmsg.oidc.AuthorizationRequest object at     |
|                     |    0x7f9420dcde10>                                  |
|                     | :sub: 'effa1d7c6fdbc66934f5af3441e0b90a...'         |
|                     | :subordinate: ['f3fcaf84322511eba12cacd...']        |
|                     | :revoked: False                                     |
|                     | :type: "ClientSessionInfo"                          |
+---------------------+-----------------------------------------------------+
| grant               | <oidcendpoint.grant.Grant object at 0x7f9420cbc240> |
+---------------------+-----------------------------------------------------+

Now if the user logs in from another client

.. code-block:: python

    from oidcmsg.oidc import AuthorizationRequest

    AUTHN_REQ_2 = AuthorizationRequest(
        state="state2",
        response_type="code",
        redirect_uri="https://client2.example.org/cb",
        scope=["openid"],
        client_id="client_2",
    )

    _pr_resp = authn_endpoint.parse_request(AUTHN_REQ_2.to_dict())
    _resp = authn_endpoint.process_request(_pr_resp)
    _code2 = _resp["response_args"]["code"]

During the process_request process there is a check if the person is
logged in (based on cookies) and if so if the authentication event is still
active if so no new authentication event is created and the user_id
gathered from the log in is passed along to the next stages which are
creating a ClientSessionInfo instance, storing it under the key
[user_id, client_id] and then together with the user (or not) create
a grant.

The former is done using the
oidcendpoint.oauth2.authorization.Authorization.setup_client_session method
the later by whatever authorization module that is named in the configuration.
Once that has been performed you can look at the result:

.. code-block:: Python

    _session_info_2 = session_manager.get_session_info_by_token(_code2)

and *_session_info_2* will have values like

+---------------------+-----------------------------------------------------+
| key                 |  value                                              |
+=====================+=====================================================+
| session_id          | diana:client_2:453c26a232e111eba998acde48001122     |
+---------------------+-----------------------------------------------------+
| client_id           | client_1                                            |
+---------------------+-----------------------------------------------------+
| user_id             | diana                                               |
+---------------------+-----------------------------------------------------+
| user_session_info   | :authentication_event:                              |
|                     |    :uid: "diana"                                    |
|                     |    :authn_info: "urn:oasis:names:tc:.."             |
|                     |    :authn_time: 1606642406                          |
|                     |    :valid_until: 1606646006                         |
|                     | :subordinate: ['client_1', 'client_2']              |
|                     | :revoked': False                                    |
|                     | :type': 'UserSessionInfo'                           |
+---------------------+-----------------------------------------------------+
| client_session_info | :authorization_request:                             |
|                     |    <oidcmsg.oidc.AuthorizationRequest object at     |
|                     |    0x7f9420dbef28>                                  |
|                     | :sub: '180d1537a71393f8471ca4d5303990b...'         |
|                     | :subordinate: ['453c26a232e111eba998acd...']        |
|                     | :revoked: False                                     |
|                     | :type: "ClientSessionInfo"                          |
+---------------------+-----------------------------------------------------+
| grant               | <oidcendpoint.grant.Grant object at 0x7f9420cbc128> |
+---------------------+-----------------------------------------------------+

As can be seen the *user_session_info* has change on one point, the set of
subordinates are two instead of one. The *authentication_event* has not changed.
Since this represents a new session there is a new ClientSessionInfo instance
representing the *client_session_info*. If you where to compare the two
client_session_infos you would see that the authorization_request is the same
but that the *sub* and *subordinate* attributes has different values.

When we now have two sessions we can look at what happens if the user wants
to do single log out. The user would trigger this by sending a request to the
session endpoint. I skip that here because at this point the involvement from
the session management subsystem is limited. It amounts to one thing and
that is to find how many clients the user has sessions with.

Given a session_id this can easily be done (and you've seen this already)
by doing:

.. code-block:: Python

    _session_info = session_manager.get_session_info_by_token(_code2)
    clients = _session_info["user_session_info"]["subordinate"]

The user is now presented with a web page where she can choose to logout
from all clients, the one she presently came in from or a subset of the
whole set. If she chose all, the next step is:

.. code-block:: Python

    res = session_endpoint.logout_all_clients(_session_info["session_id"])

res will contain information on how to do the log out from each individual
client. Whether to use front- or back channel log out and how to do it.

And that's it when it comes to the session managements involvement in
SSO and SLO.