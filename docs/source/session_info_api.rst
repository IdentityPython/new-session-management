.. _`Session Info API`:

================
Session Info API
================

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
.. _`add_subordinate`:

Adds a subordinate. If the class represents user information the
subordinates are ClientSessionInfo instances. If the class represents client
session information the subordinates are Grant instances.

remove_subordinate
++++++++++++++++++
.. _`removed_subordinate`:

Removes a subordinate or rather it removes the reference to a subordinate.
The instance the reference points to will still exist in the database.

revoke
++++++
.. _`revoke`:

This marks the SessionInfo as being revoked.

is_revoked
++++++++++
.. _`is_revoked`:

This method return True if the SessionInfo is revoked otherwise False.

