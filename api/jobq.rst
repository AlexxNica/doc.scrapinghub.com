.. _api-jobq:

========
JobQ API
========

You can use the JobQ API to retrieve finished jobs from the queue.

jobq/:project_id/list
---------------------

Lists the jobs for the specified project, in order from most recent to last.

===== =================================================
Field Description
===== =================================================
ts    The time at which the job was added to the queue.
===== =================================================

========= ============================================= ========
Parameter Description                                   Required
========= ============================================= ========
startts   UNIX timestamp at which to begin results.     No
endts     UNIX timestamp at which to end results.       No
stop      The job key at which to stop showing results. No
========= ============================================= ========

====== ==================================== ====================
Method Description                          Supported parameters
====== ==================================== ====================
GET    List jobs for the specified project. startts, endts, stop
====== ==================================== ====================

GET examples::

    $ curl -u APIKEY: https://storage.scrapinghub.com/jobq/53/list # Return jobs for a given project
    {"key":"53/7/81","ts":1397762393489}
    {"key":"53/7/80","ts":1395111612849}
    {"key":"53/7/78","ts":1393972804722}
    {"key":"53/7/77","ts":1393972734215}
    $ curl -u APIKEY: "https://storage.scrapinghub.com/jobq/53/list?startts=1359774955431&endts=1359774955440" # Return jobs finished between two timestamps
    {"key":"53/6/7","ts":1359774955439}
    {"key":"53/3/3","ts":1359774955437}
    {"key":"53/9/1","ts":1359774955431}

JobQ returns the list of jobs with the most recently finished first.

We recommend associating the key of the most recently finished job with the downloaded data. When you want to update your data later on, you can list the jobs and stop at the previously downloaded job::

    $ curl -u APIKEY: "https://storage.scrapinghub.com/jobq/53/list?stop=53/7/81" # Retrieve all jobs that have finished since the specified job
    {"key":"53/7/83","ts":1403610146780}
    {"key":"53/7/82","ts":1397827910849}
