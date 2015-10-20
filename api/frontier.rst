.. _api-frontier:

============
Frontier API
============

The *Hub Crawl Frontier* (HCF) stores pages visited and outstanding requests to
make. It can be thought of as a persistent shared storage for a crawl scheduler.

Web pages are identified by a fingerprint. This can be the URL of the page, but
crawlers may use any other string (e.g. a hash of post parameters, if it
processes post requests), so there is no requirement for the fingerprint to be
a valid URL.

A project can have many frontiers and each frontier is broken down into slots.
A separate priority queue is maintained per slot. This means that requests
from each slot can be prioritized separately and crawled at different rates and
at different times.

Arbitrary data can be stored in both the crawl queue and with the set of
fingerprints.

A typical example would be to use the URL as a fingerprint and the hostname as
a slot. The crawler should ensure that each host is only crawled from one
process at any given time so that politeness can be maintained.

Batch object
------------

======== ============================
Field    Description
======== ============================
id       Batch ID.
requests An array of request objects.
======== ============================

Request object
--------------

===== ==================================================================== ========
Field Description                                                          Required
===== ==================================================================== ========
fp    Request fingerprint.                                                 Yes
qdata Data to be stored along with the fingerprint in the request queue.   No
fdata Data to be stored along with the fingerprint in the fingerprint set. No
p     Priority: lower priority numbers are returned first. Defaults to 0.  No
===== ==================================================================== ========

/hcf/:project/:frontier/s/:slot
-------------------------------

======== ================================================
Field    Description
======== ================================================
newcount The number of new requests that have been added.
======== ================================================

Method Description                               Supported parameters
====== ========================================= ====================
POST   Enqueues a request in the specified slot. fp, qdata, fdata, p
DELETE Deletes the specified slot.
====== ========================================= ====================

POST examples::

    $ curl -u API_KEY: -d '{"fp":"/some/path.html"}'  \
        https://storage.scrapinghub.com/hcf/78/test/s/example.com
    {"newcount":1}
    # By using the same priority as request depth, the website can be traversed in breadth-first order from the starting URL.
    $ curl -u API_KEY: -d $'{"fp":"/"}\n{"fp":"page1.html", "p": 1, "qdata": {"depth": 1}}' \
        https://storage.scrapinghub.com/hcf/78/test/s/example.com
    {"newcount":2}

DELETE example::

    $ curl -u API_KEY: -X DELETE https://storage.scrapinghub.com/hcf/78/test/s/example.com/

/hcf/:project/:frontier/s/:slot/q
---------------------------------

Retrieve requests for a given slot.

========= =========================================== ========
Parameter Description                                 Required
========= =========================================== ========
mincount  The minimum number of requests to retrieve. No
========= =========================================== ========

Example::

    $ curl -u API_KEY: https://storage.scrapinghub.com/hcf/78/test/s/example.com/q
    {"id":"00013967d8af7b0001","requests":[["/",null]]}
    {"id":"01013967d8af7e0001","requests":[["page1.html",{"depth":1}]]}

/hcf/:project/:frontier/s/:slot/q/deleted
------------------------------------------

Delete a batch of requests.

Once a batch has been processed, clients should indicate that the batch is completed so that it will be removed and no longer returned when new batches are requested.

This can be achieved by posting the IDs of the completed batches::

    $ curl -u API_KEY: -d '"00013967d8af7b0001"' https://storage.scrapinghub.com/hcf/78/test/s/example.com/q/deleted

You can specify the IDs as arrays or single values. As with the previous examples, multiple lines of input is accepted.

/hcf/:project/:frontier/s/:slot/f
---------------------------------

Retrieve fingerprints for a given slot.

Example::

    $ curl -u API_KEY: https://storage.scrapinghub.com/hcf/78/test/s/example.com/f
    {"fp":"/"}
    {"fp":"page1.html"}

Results are ordered lexicographically by fingerprint value.

/hcf/:project/:frontier/list
----------------------------

Lists the frontiers for a given project.

Example::

    $ curl -u API_KEY: https://storage.scrapinghub.com/hcf/78/list
    ["test"]

/hcf/:project/:frontier/list
----------------------------

Lists the slots for a given frontier.

Example::

    $ curl -u API_KEY: https://storage.scrapinghub.com/hcf/78/test/list
    ["example.com"]


