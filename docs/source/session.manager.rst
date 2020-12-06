===============
Session Manager
===============

The Session Manager are there to managing sessions.

SessionManager
--------------

.. code-block:: python

    def __init__(
        handler: TokenHandler,
        db: Optional[object] = None,
        conf: Optional[dict] = None,
        sub_func: Optional[dict] = None):

*handler* is a TokenHandler instance. *db* may be a persistent storage module.
*conf* is session manager configuration and *sub_fuc* is a dictionary of
subject identifier functions as values and subject identifier types (public,
pairwise or ephemeral).

----

.. code-block:: Python

    def create_session(
        authn_event: AuthnEvent,
        auth_req: AuthorizationRequest,
        user_id: str,
        client_id: str = "",
        sub_type: str = "public",
        sector_identifier: str = ''):

Creates a new session. *authn_event* an AuthnEvent class instance that
describes the authentication event. *auth_req* an Authentication request.
*client_id* a client Identifier. *user_id* an user identifier.
*sub_type* provides the type of subject identifier that should be constructed.
It can either be *pairwise*, *public* or *ephemeral*. *sector_identifier*
is a sector identifier to be used when constructing a pairwise subject
identifier. If sub_type is *pairwise* then sector_identifier MUST be present.

Creating a new session means creating a
:ref:`User Session Info<UserSessionInfo>` instance and a
:ref:`Client Session Info<ClientSessionInfo>` instance and store the first
under the key *user_id* in the database and the other under the key
[*user_id*,*client_id*].

So a typical command would look like this:

.. code-block:: python

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

    user_session_info = session_manager.get(["diana"])
    client_session_info = session_manager.get(["diana", "client_1"])

**Note**: The authorization request is of course something you receive from a client.
And the authn_event does normally contain more information then this.

-----

.. code-block:: python

    def add_grant(user_id: str, client_id: str, **kwargs) -> Grant


Creates and adds a grant to a user's session.
*user_id* and *client_id* are the normal user and client identifiers.
*kwargs* are keyword arguments to the Grant class initialization.

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

-----

.. code-block:: Python

    def find_token(session_id: str, token_value: str) -> Optional[Token]

Finds a specific token belonging to a session.
*session_id* is a session identifier.
*token_value* is the value of an access/refresh token, code or some other
kind of token.

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

------

.. code-block:: Python

    def get_authentication_event(self, session_id: str) -> AuthnEvent

Finds the authentication event bound to a session.
*session_id* is a session identifier.

------

.. code-block:: Python

    def get_client_session_info(session_id: str) -> ClientSessionInfo

Returns the client session info of a session.
*session_id* is a session identifier.

------

.. code-block:: python

    def get_session_info(session_id: str) -> dict

Return a dictionary with the following keys:
    - session_id,
    - user_id,
    - client_id,
    - user_session_info,
    - client_session_info,
    - grant

All information belonging to one session.
*session_id* is a session identifier.

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


-------------------------

.. code-block:: Python

    def get_session_info_by_token(token_value: str) -> dict

Basically the same as get_session_info but here we start with
a token value rather then with a session_id.

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


--------

.. code-block:: python

    def revoke_client_session(session_id)

Revokes a client session. That is it sets the revoked attribute of
the client session info. It does not remove the client session info from the
database.
*session_id* is a session identifier.

------------

.. code-block:: python

    def revoke_grant(session_id)

Revokes a grant. That is it sets the revoked attribute of
the grant. It does not remove the grant from the
database. Nor does it revoke any of the tokens minted by the grant.
*session_id* is a session identifier.

------------

.. code-block:: python

    def revoke_token(session_id, token_value, recursive=False)

Revokes a token. That is it sets the revoked attribute of
the token. It does not remove the token from the
database. Nor does it revoke any of the tokens minted based on the token.
*session_id* is a session identifier.

------

.. code-block:: python

    def grants(session_id: str) -> List[Grant]:

Find and return all the grants that belongs to this session. This would
normally be one :ref:`Grant` instance and one or more :ref:`ExchangeGrant`
instances.
*session_id* is a session identifier.

------

.. code-block:: python

    def find_exchange_grant(
        token: str,
        resource_server: str
        ) -> Optional[Grant]:

Find a specific :ref:`ExchangeGrant` instances using a token value and the
name of a resource service the exchange grant is usable at.
