.. _`Session Info`:

===================
Session Information
===================

The SessionInfo class is the base class for ClientSessionInfo and
UserSessionInfo. The SessionInfo class itself is based on the
oidcmsg.message.Message. Beside the Message methods it also has the methods
listed below:

    - `add_subordinate`_
    - `remove_subordinate`_
    - `revoke`_
    - `is_revoked`_

add_subordinate
+++++++++++++++
.. _`info.add_subordinate`:

Adds a subordinate. If the class represents user information the
subordinates are ClientSessionInfo instances. If the class represents client
session information the subordinates are Grant instances.

remove_subordinate
++++++++++++++++++
.. _`info.removed_subordinate`:

Removes a subordinate or rather it removes the reference to a subordinate.
The instance the reference points to will still exist in the database.

revoke
++++++
.. _`info.revoke`:

This marks the SessionInfo as being revoked.

is_revoked
++++++++++
.. _`info.is_revoked`:

This method return True if the SessionInfo is revoked otherwise False.

User session information
++++++++++++++++++++++++
.. _`info.user`:

Houses the authentication event information which is the same for all session
connected to a user.
Here we also have a list of all the clients that this user has a session with.
Expressed as a dictionary this can look like this::

    {
        'authentication_event': {
            'uid': 'diana',
            'authn_info': "urn:oasis:names:tc:SAML:2.0:ac:classes:InternetProtocolPassword",
            'authn_time': 1605515787,
            'valid_until': 1605519387
        },
        'subordinate': ['client_1']
    }


Client session information
++++++++++++++++++++++++++
.. _`info.client`:

The client specific information of the session information.
Presently only the authorization request and the subject identifier (sub).
The subordinates to this set of information are the grants::

    {
        'authorization_request':{
            'client_id': 'client_1',
            'redirect_uri': 'https://example.com/cb',
            'scope': ['openid', 'research_and_scholarship'],
            'state': 'STATE',
            'response_type': ['code']
        },
        'sub': '117afe8d7bb0ace8e7fb2706034ab2d3fbf17f0fd4c949aa9c23aedd051cc9e3',
        'subordinate': ['e996c61227e711eba173acde48001122'],
        'revoked': False
    }


