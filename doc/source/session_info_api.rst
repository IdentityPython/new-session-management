.. _`Session Info API`:

================
Session Info API
================

The SessionInfo class behaves like a dictionary. It has all
the usual dictionary methods: get,set,__setitem__, __getitem__, keys, values,
items, len, __contains__. Apart from those it also has the methods listed below.

    - `add_subordinate`_
    - `remove_subordinate`_
    - `revoke`_
    - `is_revoked`_
    - `to_json`_
    - `from_json`_

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
The instance the reference points to can still exist in the database.

revoke
++++++
.. _`revoke`:

This marks the SessionInfo as being revoked.

is_revoked
++++++++++
.. _`is_revoked`:

This method return True if the SessionInfo is revoked otherwise False.

to_json
+++++++
.. _`to_json`:

Converts the SessionInfo instance to a JSON representation.

from_json
+++++++++
.. _from_json:

Sets the information in a SessionInfo instance based on the information in a
JSON object.

