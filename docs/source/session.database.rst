================
Session database
================

The model of the underlying session database. You can either
use it as it is or you can write your own implementing the same
interface.

Database
--------

.. code-block:: python

    def __init__(storage=None):

*storage* is the backend storage system. The storage system must support
a dictionary like interface with as a minimum the methods:
*__setitem__*, *__getitem__* and *get*. If no storage is provided the default
is to use an in memory dictionary.

------

.. code-block:: Python

    def get(path: List[str]) -> Union[SessionInfo, Grant]

The path can be of different length (1-3) with the following
content

    - user_id,
    - user_id and client_id or
    - user_id, client_id and grant_id

Thus if the path is of length one the response will contain
:ref:`User session info<UserSessionInfo>` . If it has the length two the response
will be :ref:`Client session information<ClientSessionInfo>` and if it is of length
three the response will be :ref:`Grant`.

------

.. code-block:: python

    def set(path: List[str], value: Union[SessionInfo, Grant])

Adds information to the database.
The same rules applies to path as for get.

------

.. code-block:: python

    def delete(path: List[str])

Removes information from the database.
The same rules applies to path as for get.
