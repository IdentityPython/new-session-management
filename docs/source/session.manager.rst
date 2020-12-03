.. _`Session Manager`:

=======================
Session Manager Package
=======================

The Session Manager Package is a library for managing sessions.

create_session
--------------
.. _mngr_create_session:

.. code-block:: Python

    create_session(authn_event: AuthnEvent,
                   auth_req: AuthorizationRequest,
                   user_id: str,
                   client_id: str = "",
                   sub_type: str = "public",
                   sector_identifier: str = ''):

Creates a new session. The create_session methods takes the following
arguments.

authn_event
    An AuthnEvent class instance that describes the authentication event.

auth_req
    The Authentication request

client_id
    The client Identifier

user_id
    The user identifier

sector_identifier
    A possible sector identifier to be used when constructing a pairwise
    subject identifier. If sub_type is *pairwise* then sector_identifier MUST
    be present.

sub_type
    The type of subject identifier that should be constructed. It can either be
    *pairwise*, *public* or *ephemeral*.

So a typical command would look like this::


    from oidcmsg.oidc import AuthorizationRequest
    from oidcendpoint.authn_event import create_authn_event

    AUTH_REQ = AuthorizationRequest(
        client_id="client_1",
        redirect_uri="https://example.com/cb",
        scope=["openid"],
        state="STATE",
        response_type="code",
    )

    authn_event = create_authn_event("diana")

    session_manager.create_session(authn_event=authn_event,
                                   auth_req=AUTH_REQ,
                                   user_id="diana",
                                   client_id="client_1",
                                   sub_type="pairwise",
                                   sector_identifier="https://example.com/sid")

The authorization request is of course something you receive from a client.
And the authn_event does normally contain more information then this.

add_grant
---------
.. _mngr_add_grant:

.. code-block:: Python

    add_grant(user_id: str, client_id: str, **kwargs) -> Grant

Creates and adds a grant to a user's session.
Method parameters:

user_id
    User identifier

client_id
    Client identifier

kwargs
    Keyword arguments to the Grant class initialization

.. code-block:: Python

    authn_event = create_authn_event('diana')
    session_manager.create_session(authn_event=authn_event,
                                   auth_req=AUTH_REQ,
                                   user_id='diana',
                                   client_id="client_1")

    grant = self.session_manager.add_grant(
        user_id="diana", client_id="client_1",
        scope=["openid", "phoe"],
        claims={"userinfo": {"given_name": None}})

find_token
----------
.. _mngr_find_token:

.. code-block:: Python

    find_token(session_id: str, token_value: str) -> Optional[Token]

Finds a specific token belonging to a session.
Method parameters:

session_id
    Session identifier

token_value
    The value of an access/refresh token, code or some other kind of token.

Code example:

.. code-block:: Python

    session_manager.create_session(authn_event=authn_event,
                                   auth_req=AUTH_REQ,
                                   user_id='diana',
                                   client_id="client_1")

    grant = session_manager.add_grant(user_id="diana",
                                      client_id="client_1")

    code = grant.mint_token("authorization_code", value="ABCD")

    _session_key = session_key('diana', 'client_1', grant.id)
    _token = self.session_manager.find_token(_session_key, code.value)

    assert _token.type == "authorization_code"
    assert _token.id == code.id



get_authentication_event
------------------------
.. _mngr_get_authentication_event:

.. code-block:: Python

    get_authentication_event(self, session_id: str) -> AuthnEvent

Finds the authentication event bound to a session.
Method parameters:

session_id
    Session identifier

get_client_session_info
-----------------------
.. _mngr_get_client_session_info:

.. code-block:: Python

    get_client_session_info(session_id: str) -> ClientSessionInfo

Returns the client session info of a session.

Method parameters:

session_id
    Session identifier

get_session_info
----------------
.. _mngr_get_session_info:

.. code-block::

    get_session_info(session_id: str) -> dict

Return a dictionary with the following keys:
    - session_id,
    - user_id,
    - client_id,
    - user_session_info,
    - client_session_info,
    - grant

All information belonging to one session.

Code example:

.. code-block:: Python

    session_manager.create_session(authn_event=authn_event,
                                   auth_req=AUTH_REQ,
                                   user_id='diana',
                                   client_id="client_1")

    grant = session_manager.add_grant(user_id="diana",
                                      client_id="client_1")

    _session_id = session_key('diana', 'client_1', grant.id)
    session_info = session_manager.get_session_info(_session_id)

    assert session_info["user_id"] == "diana"

get_session_info_by_token
-------------------------
.. _mngr_get_session_info_by_token:

Basically the same as get_session_info but here we start with
a token value rather then with a session_id.

.. code-block:: Python

    get_session_info_by_token(token_value: str) -> dict

Code example:

.. code-block:: Python

    session_manager.create_session(authn_event=authn_event,
                                   auth_req=AUTH_REQ,
                                   user_id='diana',
                                   client_id="client_1")

    grant = session_manager.add_grant(user_id="diana",
                                      client_id="client_1")

    _session_id = session_key('diana', 'client_1', grant.id)
    code = grant.mint_token(
        "authorization_code",
        value=session_manager.token_handler["code"](_session_id)
    )
    session_info = session_manager.get_session_info(_session_id)

    assert session_info["user_id"] == "diana"


revoke_client_session
---------------------
.. _mngr_revoke_client_session:

revoke_client_session(self, session_id)

revoke_grant
------------
.. _mngr_revoke_grant:

revoke_grant(self, session_id)

revoke_token
------------
.. _mngr_revoke_token:

revoke_token(self, session_id, token_value, recursive=False)

grants
------
.. _mngr_grants:

find_exchange_grant
-------------------
.. _mngr_find_exchange_grant:
