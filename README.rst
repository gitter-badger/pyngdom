
=======
pyngdom
=======

Simple python API Against Pingdom, that feature RUM extraction.

.. image:: https://travis-ci.org/Epi10/pyngdom.svg?branch=master
    :target: https://travis-ci.org/Epi10/pyngdom


Install
-------

As usual

.. code:: bash
    
    python setup.py install


Or as less ussual

.. code:: bash
    
    pip install git+https://github.com/Epi10/pyngdom


Getting Rum
-----------

If you are here is for the RUM (and that's not a bad pirate's pun).

We provide with 2 interfaces to pingdom rum, the first one its using a python-only aproach, and that will give you only
*today's* RUM information, thats good if you don't want to install extra dependencies or dont like/want selenium. for the second
approach you'll need selenium and a webbrowser, for testing, Firefox is fine, but if you really wnat to take advantage of the script
you should really use PhantomJS (the default).

Python-Only method
^^^^^^^^^^^^^^^^^^

Disclaimer, the RUM search is not part of the pingdom API, and its not supported by pingdom, so it can stop working at any time,
use under your own risk. Also this method is harder to maintain than the selenium approach, so will take longer to fix if pingdom change anything.

So how it is use:


.. code:: python

    from pyngdom import PyngdomRum
    
    pingdom = Pyngdom(
        username='user@epi10.cl',
        password='super-secret-password',
        apikey='6dz4mqdms0qaxrjstntf6myt6wz5vseg',
        account='owner@epi10.cl'
    )


Now you need to know the rum test ID, that's easy all you need to do is login into pingdom and then look for the RUM link
in the reports page, the format of the rum link is https://my.pingdom.com/rum/XXXXXXXXXXX

here is a example

.. image:: https://raw.githubusercontent.com/Epi10/pyngdom/master/docs/rumid.png
   :alt: RumID search

after you got the link all you need to do is

.. code:: python

    >>> rum = pingdom.today_rum('XXXXXXXXXXX')
    >>> print rum
    {
       u'allow_subdomains': False,
       u'total': {u'average': 2557,
            u'count': 40305,
            u'median': 1971.4791994493,
            u'p90': 4623.1101804124,
            u'p95': 5853.3586753731,
            u'p99': 10089.605}
       ....
       ....
       'url': 'http://epi10.cl'
    }
    
    
    #Get Total RUM
    >> print rum['total']
    {
        u'average': 2557,
        u'count': 40305,
        u'median': 1971.4791994493,
        u'p90': 4623.1101804124,
        u'p95': 5853.3586753731,
        u'p99': 10089.605
    }
    
    
    
    #Get RUM per geolocation that exist
    >> print rum['geo'].get('us', {})
    {
       u'average': 6657,
       u'count': 27,
       u'median': 5875.0625,
       u'p90': 9700.0416666667,
       u'p95': 10175.125,
       u'p99': 13435.125
    }
    
    # Get geographic zone (remember not all geographic zone exists)
    >> print rum['geo'].get('jp', {})
    {}



Also there is lots and lots of available information.

If you have selenium (and you should), use it.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you have selenium (and even better PhantomJS [http://phantomjs.org/]) you should use it.
This will open a new door to get RealTime RUM and its easy to implement and expand the
PyndomDriver than the normal rum, from the user point of view, both methods should be
interchangeable, but this will actually give you realtime RUM.


How do i use it?

First install selenium


.. code:: bash

    pip install selenium


Now you are ready to use it

.. code:: python

    from pyngdom import PyngdomDriver

    # If you want to use your native firefox (no extra install other than having your own firefox)

    pingdom = PyngdomDriver(
        username='user@epi10.cl',
        password='super-secret-password',
        base_driver='Firefox'
    )

    #if you have phantomjs installed

    pingdom = PyngdomDriver(
        username='user@epi10.cl',
        password='super-secret-password'
    )




then its simple, you just get the checkid of your rum (see previous section) and then call it the same


.. code:: python

    >>> rum = pingdom.today_rum('XXXXXXXXXXX')
    {
       u'allow_subdomains': False,
       u'total': {u'average': 2557,
            u'count': 40305,
            u'median': 1971.4791994493,
            u'p90': 4623.1101804124,
            u'p95': 5853.3586753731,
            u'p99': 10089.605}
       ....
       ....
       'url': 'http://epi10.cl'
    }


And now the  fun part, to get the realtime rum you just pick a sample interval (i.e. 30 seconds) and then you just


.. code:: python

    >>> rum = pingdom.realtime_rum('XXXXXXXXXXX', 30)
    #30 seconds later

    {
       u'allow_subdomains': False,
       u'total': {u'average': 2456,
            u'count': 15,
            u'median': 1971.4791994493,
            u'p90': 4623.1101804124,
            u'p95': 5853.3586753731,
            u'p99': 10089.605}
       ....
       ....
       'url': 'http://epi10.cl'
    }



And its just that simple... once again we strongly suggest using phantomjs, installit is so simple in linux and mac.


Extra API
---------

If you need the standard pingom API, i recommend using https://pypi.python.org/pypi/PingdomLib , its mature and it is
really simple to use. Never the less we include some extra functionality, using the standard pingdom API, only here
because for some projects we really need them.


.. code:: python

    from pprint import pprint

    from pyngdom import Pyngdom

    pingdom = Pyngdom(
        username='user@epi10.cl',
        password='super-secret-password',
        apikey='6dz4mqdms0qaxrjstntf6myt6wz5vseg',
        account='owner@epi10.cl'
    )

    check_list = pingdom.get_check_list()

    #print the check lists
    pprint(check_list)

    #get only the check for api.epi10.cl
    api_epi10_check = filter(lambda x: x.get('hostname') == 'api.epi10.cl', check_list.get('checks', []))[0]

    #print detailed information
    print pingdom.get_detailed_check_information(api_epi10_check['id'])
