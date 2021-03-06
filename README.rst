|Build Status| |Coverage Status| |Code Health| |PyPI Version|

Sealed Mock
===========

Whitelist the attributes/methods of your mocks instead of just letting
it create new mock objects.

`sealedmock` allows to specify when you are done defining the mock, ensuring
that any unexpected call to the mock is cached.

Sample:

.. code:: python

    from unittest.mock import Mock
    from sealedmock import seal
    m = Mock()
    m.method1.return_value.attr1.method2.return_value = 1
    seal(m)  # No new attributes can be declared
    m.method1().attr1.method2()
    # 1
    m.method1().attr2
    # Exception: AttributeError mock.method1().attr2

Sealedmock is making it into  `Python3.7 <https://github.com/python/cpython/pull/1923/>`_, but you can still use this module if you are not lucky enough to be working in Python3.7. This works even in Python2.

Install
=======

``pip install sealedmock``

Usage
=====

Given you have a file like:

.. code:: python

    import urllib2

    class SampleCodeClass(object):
        """This is sample code"""
        def calling_urlopen(self):
            return urllib2.urlopen("http://chooserandom.com")

        def calling_splithost(self):
            return urllib2.splithost("//host:port/path")

You can write a test like:

.. code:: python

    from unittest.mock import patch
    from sealedmock import seal
    @patch("tests.sample_code.urllib2")
    def test_using_decorator(mock):
            sample = sample_code.SampleCodeClass()
            mock.urlopen.return_value = 2

            seal(mock)  # No new attributes can be declared

            # calling urlopen succeeds as mock.urlopen has been defined
            sample.calling_urlopen()

            # This will fail as mock.splithost has not been defined
            sample.calling_splithost()

If you use an common Mock the second part will pass as it will create a
mock for you and return it. With sealedmock you can choose when to stop
that behaviour.

This is recursive so you can write:

.. code:: python

    @patch("sample_code.secret")
    def test_recursive(mock):
            sample = sample_code.SampleCodeClass()
            mock.secret.call1.call2.call3.return_value = 1
            seal(mock)  # No new attributes can be declared

            # If secret is not used as specified above it will fail
            # ex: if do_stuff also calls secret.call1.call9
            sample.do_stuff()

It also prevents typos on tests if used like this:

.. code:: python

    @patch("sample_code.secret")
    def test_recursive(mock):
            sample = sample_code.SampleCodeClass()

            sample.do_stuff()

            seal(mock)
            mock.asert_called_with(1)
            # Note the typo in asert (should be assert)
            # A sealed mock will rise, normal mock won't

.. |Build Status| image:: https://travis-ci.org/mariocj89/sealedmock.svg?branch=master
   :target: https://travis-ci.org/mariocj89/sealedmock
.. |Coverage Status| image:: https://coveralls.io/repos/github/mariocj89/sealedmock/badge.svg?branch=master
   :target: https://coveralls.io/github/mariocj89/sealedmock?branch=master
.. |Code Health| image:: https://landscape.io/github/mariocj89/sealedmock/master/landscape.svg?style=flat
   :target: https://landscape.io/github/mariocj89/sealedmock/master
.. |PyPI Version| image:: https://img.shields.io/pypi/v/sealedmock.svg
   :target: https://pypi.python.org/pypi/sealedmock/
