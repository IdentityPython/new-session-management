.. _`Session Manager API`:

===================
Session Manager API
===================

    - `create_session`_
    - `add_grant`_
    - `find_token`_
    - `get_authentication_event`_
    - `get_client_session_info`_
    - `get_session_info`_
    - `get_session_info_by_token`_
    - `revoke_client_session`_
    - `revoke_grant`_
    - `revoke_token`_

create_session
--------------
.. _create_session:

Creating a new session is done by running the create_session method of
the class SessionManager. The create_session methods takes the following
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


    authn_event = create_authn_event(self.user_id)
    session_manager.create_session(authn_event=authn_event, auth_req=auth_req,
                                   user_id=self.user_id, client_id=client_id,
                                   sub_type=sub_type, sector_identifier=sector_identifier)

add_grant
---------
.. _add_grant:

Creates and adds a grant to a user session.
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

    assert grant.scope == ["openid", "phoe"]

    _grant = self.session_manager.get(['diana', 'client_1', grant.id])
    assert _grant.scope == ["openid", "phoe"]

find_token
----------
.. _find_token:

Finds a specific token belonging to a session.

.. code-block:: Python

    session_manager.create_session(authn_event=authn_event,
                                   auth_req=AUTH_REQ,
                                   user_id='diana',
                                   client_id="client_1")

    grant = session_manager.add_grant(user_id="diana",
                                      client_id="client_1")

    code = grant.mint_token("authorization_code", value="ABCD")
    access_token = grant.mint_token("access_token", value="007", based_on=code)

    _session_key = session_key('diana', 'client_1', grant.id)
    _token = self.session_manager.find_token(_session_key, access_token.value)

    assert _token.type == "access_token"
    assert _token.id == access_token.id



get_authentication_event
------------------------
.. _get_authentication_event:

get_authentication_event(self, session_id)


get_client_session_info
-----------------------
.. _get_client_session_info:

get_client_session_info(self, session_id)

get_session_info
----------------
.. _get_session_info:

get_session_info(self, session_id)

get_session_info_by_token
-------------------------
.. _get_session_info_by_token:

get_session_info_by_token(self, token_value)

revoke_client_session
---------------------
.. _revoke_client_session:

revoke_client_session(self, session_id)

revoke_grant
------------
.. _revoke_grant:

revoke_grant(self, session_id)

revoke_token
------------
.. _revoke_token:

revoke_token(self, session_id, token_value, recursive=False)