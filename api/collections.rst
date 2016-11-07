.. _api-collections:

===============
Collections API
===============

Scrapinghub's *Collections* are key-value stores for arbitrary large
number of records. They are especially useful to store information
produced and/or used by multiple scraping jobs.

*Note that If you want to store urls to be processed by one or multiple jobs,
please check the frontier api.*

A record can be any json dictionnary. They are identified by a ``_key`` field.


Quickstart
==========

A collection is identified by a *project id*, a *type* and a *name*.


Create/Update a record:
***********************

    $ curl -u $APIKEY: -X POST -d '{"_key": "foo", "value": "bar"}' \
        https://storage.scrapinghub.com/collections/78/s/my_collection


Access a record:
****************

    $ curl -u $APIKEY: -X GET \
        https://storage.scrapinghub.com/collections/78/s/my_collection/foo


Delete a record:
****************

    $ curl -u $APIKEY: -X DELETE \
        https://storage.scrapinghub.com/collections/78/s/my_collection/foo


List records:
*************

    $ curl -u $APIKEY: -X GET \
        https://storage.scrapinghub.com/collections/78/s/my_collection


Create/Update multiple records:
*******************************

We use the jsonline format by default (json objects separated by a newline):

    $ curl -u $APIKEY: -X POST -d '{"_key": "foo", "value": "bar"}\n{"_key": "goo", "value": "baz"}' \
        https://storage.scrapinghub.com/collections/78/s/my_collection


Details
=======

Types
-----

The following collection types are available:

====  ===================== ========================== ================================================================
Type  Full name             Hubstorage method          Description
====  ===================== ========================== ================================================================
s     store                 new_store                  Basic set store
cs    cached store          new_cached_store           Items expire after a month
vs    versioned store       new_versioned_store        Up to 3 copies of each item will be retained
vcs   versioned cache store new_versioned_cached_store Multiple copies are retained, and each one expires after a month
====  ===================== ========================== ================================================================


Constraints
-----------

- Records are limited to a serialized size of ``10MO``;
- Javascript's ``inf`` values are not supported;
- Floating-point number can't be larger than 2^64 - 1.


API
===

collections/:project_id/:type/:collection
-----------------------------------------

Read or write items from or to a collection.

=========== ========================================================= ========
Parameter   Description                                               Required
=========== ========================================================= ========
key         Read items with specified key. Multiple values supported. No
prefix      Read items with specified key prefix.                     No
prefixcount Maximum number of values to return per prefix.            No
startts     UNIX timestamp at which to begin results.                 No
endts       UNIX timestamp at which to end results.                   No
=========== ========================================================= ========

====== ========================================= ========================================
Method Description                               Supported parameters
====== ========================================= ========================================
GET    Read items from the specified collection. key, prefix, prefixcount, startts, endts
POST   Write items to the specified collection.
====== ========================================= ========================================

.. note:: Pagination and meta parameters are supported, see :ref:`api-overview-pagination` and :ref:`api-overview-metapar`.

GET examples::

    $ curl -u APIKEY: "https://storage.scrapinghub.com/collections/78/s/my_collection?key=foo1&key=foo2"
    {"value":"bar1"}
    {"value":"bar2"}
    $ curl -u APIKEY: https://storage.scrapinghub.com/collections/78/s/my_collection?prefix=f
    {"value":"bar"}
    $ curl -u APIKEY: "https://storage.scrapinghub.com/collections/78/s/my_collection?startts=1402699941000&endts=1403039369570"
    {"value":"bar"}

.. note:: When using :ref:`python-hubstorage <api-overview-ep-storage>`, you should use the method ``iter_json`` to iterate through items in order to filter them.

Prefix filters, unlike other filters, use indexes and should be used
when possible. You can use the ``prefixcount`` parameter to limit the
number of values returned for each prefix.

A common pattern is to download changes within a certain time period.
You can use the ``startts`` and ``endts`` parameters to select records
within a certain time window.

The current timestamp can be retrieved like so::

    $ curl https://storage.scrapinghub.com/system/ts
    1403039369570

.. note:: Timestamp filters may perform poorly when selecting a small number of records from a large collection.


collections/:project_id/:type/:collection/:item
-----------------------------------------------

Read an individual item.

HTTP::

    $ curl -u APIKEY: https://storage.scrapinghub.com/collections/78/s/my_collection/foo
    {"value":"bar"}

Python (:ref:`python-hubstorage<api-overview-ep-storage>`)::

    >>> store = project.collections.new_store('my_collection')
    >>> store.get('foo')
    {u'value': u'bar'}

collections/:project_id/:type/:collection/:item/value
-----------------------------------------------------

Read an individual item value.

GET example::

    $ curl -u APIKEY: https://storage.scrapinghub.com/collections/78/s/my_collection/foo/value
    bar

