.. _update_by_query:

Update By Query
================

The ``Update By Query`` object
-------------------------------

The ``Update By Query`` object enables the use of the 
`_update_by_query <https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html>`_
endpoint to perform an update on documents that match a search query.

The object is implemented as a modification of the ``Search`` object, containing a 
subset of its query methods, as well as a script method, which is used to make updates.

The ``Update By Query`` object implements the following ``Search`` query types:
  
  * queries

  * filters

  * excludes

For more information on queries, see the :ref:`search_dsl` chapter.

Like the ``Search`` object, the API is designed to be chainable. This means that the ``Update By Query`` object
is immutable: all changes to the object will result in a shallow copy being created which
contains the changes. This means you can safely pass the ``Update By Query`` object to
foreign code without fear of it modifying your objects as long as it sticks to
the ``Update By Query`` object APIs.

You can pass an instance of the low-level `elasticsearch client <https://elasticsearch-py.readthedocs.io/>`_ when
instantiating the ``Update By Query`` object:

.. code:: python

    from elasticsearch import Elasticsearch
    from elasticsearch_dsl import UpdateByQuery

    client = Elasticsearch()

    ubq = UpdateByQuery(using=client)

You can also define the client at a later time (for more options see the
:ref:`configuration` chapter):

.. code:: python

    ubq = ubq.using(client)

.. note::

    All methods return a *copy* of the object, making it safe to pass to
    outside code.

The API is chainable, allowing you to combine multiple method calls in one
statement:

.. code:: python

    ubq = UpdateByQuery().using(client).query("match", title="python")

To send the request to Elasticsearch:

.. code:: python

    response = ubq.execute()

It should be noted, that there are limits to the chaining using the script method: calling script multiple times will 
overwrite the previous value. That is, only a single script can be sent with a call. An attempt to use two scripts will 
result in only the second script being stored.

Given the below example:

.. code:: python
	
	ubq = UpdateByQuery().using(client).script(source="ctx._source.likes++").script(source="ctx._source.likes+=2")

This means that the stored script by this client will be ``'source': 'ctx._source.likes+=2'`` and the previous call
will not be stored.

For debugging purposes you can serialize the ``Update By Query`` object to a ``dict``
explicitly:

.. code:: python

    print(ubq.to_dict())

Serialization and Deserialization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The search object can be serialized into a dictionary by using the
``.to_dict()`` method.

You can also create a ``Update By Query`` object from a ``dict`` using the ``from_dict``
class method. This will create a new ``Update By Query`` object and populate it using
the data from the dict:

.. code:: python

  ubq = UpdateByQuery.from_dict({"query": {"match": {"title": "python"}}})

If you wish to modify an existing ``Update By Query`` object, overriding it'ubq
properties, instead use the ``update_from_dict`` method that alters an instance
**in-place**:

.. code:: python

  ubq = UpdateByQuery(index='i')
  ubq.update_from_dict({"query": {"match": {"title": "python"}}, "size": 42})

Extra properties and parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To set extra properties of the search request, use the ``.extra()`` method.
This can be used to define keys in the body that cannot be defined via a
specific API method like ``explain``:

.. code:: python

  ubq = ubq.extra(explain=True)

To set query parameters, use the ``.params()`` method:

.. code:: python

  ubq = ubq.params(routing="42")

Response
--------

You can execute your search by calling the ``.execute()`` method that will return
a ``Response`` object. The ``Response`` object allows you access to any key
from the response dictionary via attribute access. It also provides some
convenient helpers:

.. code:: python

  response = ubq.execute()

  print(response.success())
  # True

  print(response.took)
  # 12

If you want to inspect the contents of the ``response`` objects, just use its
``to_dict`` method to get access to the raw data for pretty printing.
