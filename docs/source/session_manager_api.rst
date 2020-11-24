.. _`Session Manager API`:

===================
Session Manager API
===================

    - `create_session`_
    - `add_grant`_
    - `find_token`_
    - `get_authentication_event`_
    - `get_client_session_info`_
    - `get_grant_by_response_type`_
    - `get_session_info`_
    - `get_session_info_by_token`_
    - `get_sids_by_user_id`_
    - `get_user_info`_
    - `grants`_
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
    subject identifier

sub_type
    The type of subject identifier that should be constructed. It can either be
    *pairwise* or *public*.

So a typical command would look like this::


    authn_event = create_authn_event(self.user_id)
    session_manager.create_session(authn_event=authn_event, auth_req=auth_req,
                                   user_id=self.user_id, client_id=client_id,
                                   sub_type=sub_type, sector_identifier=sector_identifier)

add_grant
---------
.. _add_grant:

add_grant(self, user_id, client_id, **kwargs)

find_token
----------
.. _find_token:

find_token(self, session_id, token_value)

get_authentication_event
------------------------
.. _get_authentication_event:

get_authentication_event(self, session_id)


get_client_session_info
-----------------------
.. _get_client_session_info:

get_client_session_info(self, session_id)

get_grant_by_response_type
--------------------------
.. _get_grant_by_response_type:

get_grant_by_response_type(self, user_id, client_id)

get_session_info
----------------
.. _get_session_info:

get_session_info(self, session_id)

get_session_info_by_token
-------------------------
.. _get_session_info_by_token:

get_session_info_by_token(self, token_value)

get_sids_by_user_id
-------------------
.. _get_sids_by_user_id:

get_sids_by_user_id(self, user_id)

get_user_info
-------------
.. _get_user_info:

get_user_info(self, uid)

grants
------
.. _grants:

grants(self, session_id)

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