+++
title = "Recovering from HTTP errors using URL handlers"
date = 2014-02-28T07:44:00Z
[taxonomies]
categories = ["articles", "recipe"]
tags = ["python", "http", "url", "error", "handling", "recovery", "explained"]
+++
_This article shows how URL handers, defined by urllib2, can be employed in practice in order to circumvent troubles we usually find when we write robots for collecting information from the Internet._

First things first (and usually a source of confusion): There are two sister libraries in Python which address retrieval of information from URLs; they are: ``urllib`` and ``urllib2``. Conceptually, ``urllib2`` works as a derived class of ``urllib``. Just conceptually, because the actual implementation does not employ classes as a conventional object oriented paradigm would suggest.

If you are seeking detailed documentation about these libraries, I'm afraid to inform that your only choice is spending a couple of hours studying the source code of [urllib.py]https://bitbucket.org/mirror/cpython/src/6017d19669c3f34d9366ce60b8eb8b64054134ae/Lib/urllib.py?at=2.7) and [urllib2.py](https://bitbucket.org/mirror/cpython/src/6017d19669c3f34d9366ce60b8eb8b64054134ae/Lib/urllib2.py?at=2.7).


## Setting an User Agent

OK. Now that you have the full documentation at hand... I mean: you followed the links above and you are reading the source code... then, we can start. The first thing our robot needs to do is hiding its presence from the server side. One simple measure is employing an innocent user agent. We need to define a class derived from ``urllib2.BaseHandler`` which is responsible for setting the user agent before a request is sent to the server side. This is shown below:
```python
import urllib2

class UserAgentProcessor(urllib2.BaseHandler):
    """A handler to add a custom UA string to urllib2 requests
    """
    def __init__(self, uastring):
        self.handler_order = 100
        self.ua = uastring

    def http_request(self, request):
        request.add_header("User-Agent", self.ua)
        return request

    https_request = http_request
```

(Credits: This code was shamelessly copied from an [article by Andrew Rowls](http://techknack.net/python-urllib2-handlers/))


## Handling HTTP ERROR 404 (Not Found)

There are other things we need to do, such as throttling our requests, otherwise the server side will easily guess that there's a robot on our side sending dozens of requests per second. But throttling is a subject that I'm not going to cover here. You can later create your throttling handler, after you get better acquainted with some techniques covered in this article.

Some webservers are really busy, which may cause failures to our requests. Other webservers deliberatly reject requests given certain circumstances, for example: the server side may detect that we are sending dozens of requests per second and may decide to punish us for 10 minutes. Again we are back to the subject of throttling, which we are not going to cover here. But let's address this sort of issue partially, which may be of practical use in a majority of situations.

Let's say the webserver responds HTTP ERROR 404 (Not Found) eventually (or even regularly), even when the resource is existing in reality. We just need to be a little skeptic and send another request after waiting a couple of seconds. Eventually we need to be far more skeptic (or a little stubborn, if you will) and send several additional requests, before we become sure enough that the resource is actually and truly non-existent.

What we need to do is basically stamp requests so that we will have means to determine whether a request needs to be sent again to the server side, eventually waiting some time before that. Also, requests to different webservers may require different parameters for number of retries and for the delay to be employed. See below how we implemented this things:

```python
import urllib2


class HTTPNotFoundHandler(urllib2.BaseHandler):
    """A handler which retries access to resources when 404 (NotFound) is received
    """

    handler_order = 600 # before HTTPDigestAuthHandler and ProxyDigestAuthHandler

    def __init__(self, retries=5, delay=2):
        self.retries = int(retries)
        self.delay   = float(delay)
        assert(self.retries >= 1)
        assert(self.delay >= 0.0)

    def http_request(self, req):
        if hasattr(req, 'headers') and 'Error_404' in req.headers:
            Error_404 = req.headers['Error_404']
            assert(int(Error_404['retries']) >= 1)
            assert(float(Error_404['delay']) >= 0.0)
        return req

    def http_error_404(self, req, fp, code, msg, headers):
        if hasattr(req, 'headers') and 'Error_404' in req.headers:
            Error_404 = req.headers['Error_404']
        else:
            Error_404 = dict()
            Error_404['delay']   = self.delay
            Error_404['retries'] = self.retries

        count   = Error_404['count'] if 'count' in Error_404 else 1
        retries = Error_404['retries']
        delay   = Error_404['delay']
        if count == retries:
            raise urllib2.HTTPError(req.get_full_url(),
                                    code,
                                    msg,
                                    headers,
                                    fp)
        else:
            # Don't close the fp until we are sure that

            # we won't use it with HTTPError.
            fp.read()
            fp.close()
            # sleep a little while
            from time import sleep
            sleep(delay)
            # send another request
            Error_404['count'] = count + 1
            req.add_header('Error_404', Error_404)
            return self.parent.open(req)

    https_error_404 = http_error_404
```

Now, let's add two utility functions:

```python
def install_opener(opener=None):
    import urllib2
    if opener is None:
        urllib2.install_opener(build_opener())
    else:
        urllib2.install_opener(opener)
    return urllib2

def build_opener(

                  user_agent='Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:24.0) Gecko/20100101 Firefox/24.0',
                  http_404_retries=3,
                  http_404_delay=2.0):
    return urllib2.build_opener(
        UserAgentProcessor(user_agent),
        HTTPNotFoundHandler(http_404_retries, http_404_delay) )
```

Just put all the code you see in this up to this point into a file, say: ``api.py``.

## Test cases

Now, let's  create some test cases for it, using ``pytest``. First thing consists on creating a ``conftest.py`` file, like shown below:

```python
from __future__ import print_function

from pytest import fixture

@fixture
def opener():
    from mypackage.api import api
    return api.build_opener()

@fixture
def urllib2(opener):
    from mypackage.api import api
    return api.install_opener(opener)
```

If you are not acquainted to ``pytest``, a very brief explanation of the code above is that we are defining functions ``opener`` and ``urllib2`` which we will later employ as parameters to other functions. In a nutshell, ``pytest`` replaces the parameter by a call to the special functions (marked by ``@fixture``) we have defined.

Now, let's create a file for test cases called test_urllib.py, like shown below:

```python
import pytest

class TestOpeners(object):

    def xtest_build_opener(self, opener):
        pass

    def xtest_existing(self, urllib2):
        url = 'http://google.com'
        f = urllib2.urlopen(url)
        assert(f.code == 200)

    def xtest_existing_but_faulty(self, urllib2):
        url = 'http://biz.yahoo.com/p/'
        f = urllib2.urlopen(url)
        assert(f.code == 200)

    def xtest_non_existing(self, urllib2):
        from urllib2 import HTTPError
        url = 'http://google.com/this_url_does_not_exist'
        with pytest.raises(HTTPError):
            f = urllib2.urlopen(url)

    def test_non_existing_with_header(self, urllib2):
        from urllib2 import HTTPError
        url = 'http://google.com/this_url_does_not_exist'
        req = urllib2.Request(url, headers = {
            'Error_404'  : { 'retries': 5,
                             'delay'  : 2.0 }})
        with pytest.raises(HTTPError):
            f = urllib2.urlopen(req)

    def test_wrong_header_retries_1(self, urllib2):
        from urllib2 import HTTPError
        url = 'http://google.com'
        req = urllib2.Request(url, headers = {
            'Error_404' : { 'retries': 'rubbish',
                            'delay'  : 2.0 }})
        with pytest.raises(ValueError):
            f = urllib2.urlopen(req)

    def test_wrong_header_retries_2(self, urllib2):
        from urllib2 import HTTPError
        url = 'http://google.com/this_url_does_not_exist'
        req = urllib2.Request(url, headers = {
            'Error_404' : { 'retries': 0,
                            'delay'  : 2.0 }})
        with pytest.raises(AssertionError):
            f = urllib2.urlopen(req)
```

## Conclusion

You can have better and more robust control of requests without even touching your application code by installing a custom opener to ``urllib2``.

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
