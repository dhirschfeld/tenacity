Tenacity
========
.. image:: https://img.shields.io/pypi/v/tenacity.svg
    :target: https://pypi.python.org/pypi/tenacity

.. image:: https://img.shields.io/travis/jd/tenacity.svg
    :target: https://travis-ci.org/jd/tenacity

.. image:: https://img.shields.io/pypi/dm/tenacity.svg
    :target: https://pypi.python.org/pypi/tenacity

Tenacity is an Apache 2.0 licensed general-purpose retrying library, written in
Python, to simplify the task of adding retry behavior to just about anything.
It originates from a fork of `Retrying`_

.. _Retrying: https://github.com/rholder/retrying

The simplest use case is retrying a flaky function whenever an `Exception`
occurs until a value is returned.

.. code-block:: python

    import random
    from tenacity import retry

    @retry
    def do_something_unreliable():
        if random.randint(0, 10) > 1:
            raise IOError("Broken sauce, everything is hosed!!!111one")
        else:
            return "Awesome sauce!"

    print do_something_unreliable()


Features
--------

- Generic Decorator API
- Specify stop condition (i.e. limit by number of attempts)
- Specify wait condition (i.e. exponential backoff sleeping between attempts)
- Customize retrying on Exceptions
- Customize retrying on expected returned result


Installation
------------

To install *tenacity*, simply:

.. code-block:: bash

    $ pip install tenacity


Examples
----------

As you saw above, the default behavior is to retry forever without waiting.

.. code-block:: python

    @retry
    def never_give_up_never_surrender():
        print "Retry forever ignoring Exceptions, don't wait between retries"

Let's be a little less persistent and set some boundaries, such as the number
of attempts before giving up.

.. code-block:: python

    @retry(stop=stop_after_attempt(7))
    def stop_after_7_attempts():
        print "Stopping after 7 attempts"

We don't have all day, so let's set a boundary for how long we should be
retrying stuff.

.. code-block:: python

    @retry(stop=stop_after_delay(10000))
    def stop_after_10_s():
        print "Stopping after 10 seconds"

Most things don't like to be polled as fast as possible, so let's just wait 2
seconds between retries.

.. code-block:: python

    @retry(wait=wait_fixed(2000))
    def wait_2_s():
        print "Wait 2 second between retries"


Some things perform best with a bit of randomness injected.

.. code-block:: python

    @retry(wait=wait_random(min=1000, max=2000))
    def wait_random_1_to_2_s():
        print "Randomly wait 1 to 2 seconds between retries"

Then again, it's hard to beat exponential backoff when retrying distributed
services and other remote endpoints.

.. code-block:: python

    @retry(wait=wait_exponential(multiplier=1000, max=10000))
    def wait_exponential_1000():
        print "Wait 2^x * 1000 milliseconds between each retry, up to 10 seconds, then 10 seconds afterwards"


Then again, it's hard to beat exponential backoff when retrying distributed
services and other remote endpoints.

.. code-block:: python

    @retry(wait=wait_combine(wait_fixed(3), wait_jitter(2)))
    def wait_fixed_jitter():
        print "Wait at least 3 seconds, and add up to 2 seconds of random delay"


We have a few options for dealing with retries that raise specific or general
exceptions, as in the cases here.

.. code-block:: python

    @retry(retry=retry_if_exception(IOError))
    def might_io_error():
        print "Retry forever with no wait if an IOError occurs, raise any other errors"

    @retry(retry_on_exception=retry_if_io_error, wrap_exception=True)
    def only_raise_retry_error_when_not_io_error():
        print "Retry forever with no wait if an IOError occurs, raise any other errors wrapped in RetryError"

We can also use the result of the function to alter the behavior of retrying.

.. code-block:: python

    def is_none_p(value):
        """Return True if value is None"""
        return value is None

    @retry(retry=retry_if_result(is_none_p))
    def might_return_none():
        print "Retry forever ignoring Exceptions with no wait if return value is None"

We can also combine several conditions:

.. code-block:: python

    def is_none_p(value):
        """Return True if value is None"""
        return value is None

    @retry(retry=retry_any(retry_if_result(is_none_p), retry_if_exception()))
    def might_return_none():
        print "Retry forever ignoring Exceptions with no wait if return value is None"

Any combination of stop, wait, etc. is also supported to give you the freedom
to mix and match.

Contribute
----------

#. Check for open issues or open a fresh issue to start a discussion around a
   feature idea or a bug.
#. Fork `the repository`_ on GitHub to start making your changes to the
   **master** branch (or branch off of it).
#. Write a test which shows that the bug was fixed or that the feature works as
   expected.

.. _`the repository`: https://github.com/jd/tenacity