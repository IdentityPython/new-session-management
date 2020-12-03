================
Session database
================

The model of the underlying session database. You can either
use it as it is or you can write your own implementing the same
interface.

The class has the following methods:

get
---

.. code-block:: Python

    get(path: List[str]) -> Union[SessionInfo, Grant]

The path can be of different length (1-3) with the following
content

    - user_id,
    - user_id and client_id or
    - user_id, client_id and grant_id

Thus if the path is of length one the response will contain
:ref:`User session info<info.user>` . If it has the length two the response
will be :ref:`Client session information<info.client>` and if it is of length
three the response will be :ref:`Grant information`.

set
---

.. code-block:: python

    set(path: List[str], value: Union[SessionInfo, Grant])

Adds information to the database.

The same rules applies to path as for get.

delete
------

.. code-block:: python

    delete(path: List[str])

Removes information from the database

The same rules applies to path as for get.
