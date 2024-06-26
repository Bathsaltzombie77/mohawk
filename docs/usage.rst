.. _usage:

============
Using Mohawk
============

There are two parties involved in `Hawk`_ communication: a
:class:`sender <mohawk.Sender>` and a :class:`receiver <mohawk.Receiver>`.
<<<<<<< HEAD
They use a shared secret to sign and verify each other's messages.

**Sender**
    A client who wants to access a Hawk-protected resource.
    The client will sign their request and upon
    receiving a response will also verify the response signature.
=======
They use use a shared secret to sign and verify each other's messages.

**Sender**
    A client who wants to access a Hawk-protected resource.
    The client will sign their request and when they receive a
    response they will also verify the response signature.
>>>>>>> 14f3100... Document all the things

**Receiver**
    A server that uses Hawk to protect its resources. The server will check
    the signature of an incoming request before accepting it. It also signs
    its response using the same shared secret.

What are some good use cases for Hawk? This library was built for the case of
securing API connections between two back-end servers. Hawk is a good
fit for this because you can keep the shared secret safe on each machine.
Hawk may not be a good fit for scenarios where you can't protect the shared
secret.

After getting familiar with usage, you may want to consult the :ref:`security`
section.

.. testsetup:: usage

<<<<<<< HEAD
    class Requests:
        def post(self, *a, **kw): pass
    requests = Requests()

    credentials = {'id': 'some-sender',
                   'key': 'a long, complicated secret',
=======
    class Request:
        def post(self, *a, **kw): pass
    request = Request()

    credentials = {'id': 'some-sender',
                   'key': 'some complicated SEKRET',
>>>>>>> 14f3100... Document all the things
                   'algorithm': 'sha256'}
    allowed_senders = {}
    allowed_senders['some-sender'] = credentials

    class Memcache:
        def get(self, *a, **kw):
            return False
        def set(self, *a, **kw): pass
    memcache = Memcache()

<<<<<<< HEAD
.. _`sending-request`:

=======
>>>>>>> 14f3100... Document all the things
Sending a request
=================

Let's say you want to make an HTTP request like this:

.. doctest:: usage

    >>> url = 'https://some-service.net/system'
    >>> method = 'POST'
    >>> content = 'one=1&two=2'
    >>> content_type = 'application/x-www-form-urlencoded'

Set up your Hawk request by creating a :class:`mohawk.Sender` object
with all the elements of the request that you need to sign:

.. doctest:: usage

    >>> from mohawk import Sender
    >>> sender = Sender({'id': 'some-sender',
<<<<<<< HEAD
    ...                  'key': 'a long, complicated secret',
=======
    ...                  'key': 'some complicated SEKRET',
>>>>>>> 14f3100... Document all the things
    ...                  'algorithm': 'sha256'},
    ...                 url,
    ...                 method,
    ...                 content=content,
    ...                 content_type=content_type)

This provides you with a Hawk ``Authorization`` header to send along
with your request:

.. doctest:: usage

    >>> sender.request_header
    'Hawk mac="...", hash="...", id="some-sender", ts="...", nonce="..."'

Using the `requests`_ library just as an example, you would send your POST
like this:

<<<<<<< HEAD
.. doctest:: usage

    >>> requests.post(url, data=content,
    ...               headers={'Authorization': sender.request_header,
    ...                        'Content-Type': content_type})

Notice how both the content and content-type values were signed by the Sender.
In the case of a GET request you'll probably need to sign empty strings like
``Sender(..., 'GET', content='', content_type='')``,
=======
 .. doctest:: usage

    >>> request.post(url, data=content,
    ...              headers={'Authorization': sender.request_header,
    ...                       'Content-Type': content_type})

Notice how both the content and content-type values were signed by the Sender.
In the case of a GET request you'll probably need to sign empty strings like
``Sender(..., content='', content_type='')``,
>>>>>>> 14f3100... Document all the things
that is, if your request library doesn't
automatically set a content-type for GET requests.

If you only intend to work with :class:`mohawk.Sender`,
skip down to :ref:`verify-response`.

.. _`receiving-request`:

Receiving a request
===================

On the receiving end, such as a web server, you'll need to set up a
:class:`mohawk.Receiver` object to accept and respond to
:class:`mohawk.Sender` requests.

First, you need to give the receiver a callable that it can use to look
up sender credentials:

.. doctest:: usage

    >>> def lookup_credentials(sender_id):
    ...     if sender_id in allowed_senders:
    ...         # Return a credentials dictionary formatted like the sender example.
    ...         return allowed_senders[sender_id]
    ...     else:
    ...         raise LookupError('unknown sender')

An incoming request will probably arrive in an object like this,
depending on your web server framework:

.. doctest:: usage

    >>> request = {'headers': {'Authorization': sender.request_header,
    ...                        'Content-Type': content_type},
    ...            'url': url,
    ...            'method': method,
    ...            'content': content}

Create a :class:`mohawk.Receiver` using values from the incoming request:

.. doctest:: usage

    >>> from mohawk import Receiver
    >>> receiver = Receiver(lookup_credentials,
    ...                     request['headers']['Authorization'],
    ...                     request['url'],
    ...                     request['method'],
    ...                     content=request['content'],
    ...                     content_type=request['headers']['Content-Type'])

If this constructor does not raise any :ref:`exceptions` then the signature of
the request is correct and you can proceed.

<<<<<<< HEAD
.. important::

    The server running :class:`mohawk.Receiver` code should synchronize its
    clock with something like `TLSdate`_ to make sure it compares timestamps
    correctly.

=======
>>>>>>> 14f3100... Document all the things
Responding to a request
=======================

It's optional per the `Hawk`_ spec but a :class:`mohawk.Receiver`
should sign its response back to the client to prevent certain attacks.

The receiver starts by building a message it wants to respond with:

.. doctest:: usage

    >>> response_content = '{"msg": "Hello, dear friend"}'
    >>> response_content_type = 'application/json'
    >>> header = receiver.respond(content=response_content,
    ...                           content_type=response_content_type)

This provides you with a similar Hawk header to use in the response:

.. doctest:: usage

    >>> receiver.response_header
    'Hawk mac="...", hash="...="'

Using your web server's framework, respond with a
<<<<<<< HEAD
``Server-Authorization`` header. For example:
=======
``Server-Authorization`` header, something like this:
>>>>>>> 14f3100... Document all the things

.. doctest:: usage

    >>> response = {
    ...     'headers': {'Server-Authorization': receiver.response_header,
    ...                 'Content-Type': response_content_type},
    ...     'content': response_content
    ... }

.. _`verify-response`:

Verifying a response
====================

When the :class:`mohawk.Sender`
receives a response it should verify the signature to
make sure nothing has been tampered with:

.. doctest:: usage

    >>> sender.accept_response(response['headers']['Server-Authorization'],
    ...                        content=response['content'],
    ...                        content_type=response['headers']['Content-Type'])


If this method does not raise any :ref:`exceptions` then the signature of
the response is correct and you can proceed.

<<<<<<< HEAD
Allowing senders to adjust their timestamps
===========================================

The easiest way to avoid timestamp problems is to synchronize your
server clock using something like `TLSdate`_.

If a sender's clock is out of sync with the receiver, its message might
expire prematurely. In this case the receiver should respond with a header
the sender can use to adjust its timestamp.

When receiving a request you might get a :class:`mohawk.exc.TokenExpired`
exception. You can access the ``www_authenticate`` property on the
exception object to respond correctly like this:

.. doctest:: usage
    :hide:

    >>> exp_sender = Sender({'id': 'some-sender',
    ...                      'key': 'a long, complicated secret',
    ...                      'algorithm': 'sha256'},
    ...                     url,
    ...                     method,
    ...                     content=content,
    ...                     content_type=content_type,
    ...                     _timestamp=1)
    >>> request['headers']['Authorization'] = exp_sender.request_header

.. doctest:: usage

    >>> from mohawk.exc import TokenExpired
    >>> try:
    ...     receiver = Receiver(lookup_credentials,
    ...                         request['headers']['Authorization'],
    ...                         request['url'],
    ...                         request['method'],
    ...                         content=request['content'],
    ...                         content_type=request['headers']['Content-Type'])
    ... except TokenExpired as expiry:
    ...     response['headers']['WWW-Authenticate'] = expiry.www_authenticate
    ...     print(expiry.www_authenticate)
    Hawk ts="...", tsm="...", error="token with UTC timestamp...has expired..."

.. doctest:: usage
    :hide:

    >>> request['headers']['Authorization'] = sender.request_header

A compliant client can look for this response header and parse the
``ts`` property (the server's "now" timestamp) and
the ``tsm`` property (a MAC calculation of ``ts``). It can then recalculate the
MAC using its own credentials and if the MACs both match it can trust that this
is the real server's timestamp. This allows the sender to retry the request
with an adjusted timestamp.

=======
>>>>>>> 14f3100... Document all the things
.. _nonce:

Using a nonce to prevent replay attacks
=======================================

A replay attack is when someone copies a Hawk authorized message and
re-sends the message without altering it.
Because the Hawk signature would still be valid, the receiver may
accept the message. This could have unintended side effects such as increasing
the quantity of an item just purchased if it were a commerce API that had an
``increment-item`` service.

Hawk protects against replay attacks in a couple ways. First, a receiver checks
<<<<<<< HEAD
the timestamp of the message which may result in a
:class:`mohawk.exc.TokenExpired` exception.
Second, every message includes a `cryptographic nonce`_
which is a unique
identifier. In combination with the sender's id and the request's timestamp, a
receiver can use the nonce to know if it has *already* received the request. If
so, the :class:`mohawk.exc.AlreadyProcessed` exception is raised.
=======
the timestamp of the message and if it has expired then it will reject the
message. Second, every message includes a `cryptographic nonce`_
which is a unique
identifier. In combination with the timestamp, a receiver can use the nonce to
know if it has *already* received the request. If so, it will reject it.
>>>>>>> 14f3100... Document all the things

By default, Mohawk doesn't know how to check nonce values; this is something
your application needs to do.

.. important::

    If you don't configure nonce checking, your application could be
    susceptible to replay attacks.

<<<<<<< HEAD
Make a callable that returns True if a sender's nonce plus its timestamp has been
seen already. Here is an example using something like memcache:

.. doctest:: usage

    >>> def seen_nonce(sender_id, nonce, timestamp):
    ...     key = '{id}:{nonce}:{ts}'.format(id=sender_id, nonce=nonce,
    ...                                      ts=timestamp)
=======
Make a callable that returns True if a nonce and timestamp have been
seen or not. Here is an example using something like memcache:

.. doctest:: usage

    >>> def seen_nonce(nonce, timestamp):
    ...     key = '{n}:{ts}'.format(n=nonce, ts=timestamp)
>>>>>>> 14f3100... Document all the things
    ...     if memcache.get(key):
    ...         # We have already processed this nonce + timestamp.
    ...         return True
    ...     else:
    ...         # Save this nonce + timestamp for later.
    ...         memcache.set(key, True)
    ...         return False

Because messages will expire after a short time you don't need to store
nonces for much longer than that timeout. See :class:`mohawk.Receiver`
for the default timeout.

Pass your callable as a ``seen_nonce`` argument to :class:`mohawk.Receiver`:

.. doctest:: usage

    >>> receiver = Receiver(lookup_credentials,
    ...                     request['headers']['Authorization'],
    ...                     request['url'],
    ...                     request['method'],
    ...                     content=request['content'],
    ...                     content_type=request['headers']['Content-Type'],
    ...                     seen_nonce=seen_nonce)

If ``seen_nonce()`` returns True, :class:`mohawk.exc.AlreadyProcessed`
will be raised.

When a *sender* calls :meth:`mohawk.Sender.accept_response`, it will receive
a Hawk message but the nonce will be that of the original request.
<<<<<<< HEAD
In other words, the nonce received is the same nonce that the sender
generated and signed when initiating the request.
This generally means you don't have to worry about *response* replay attacks.
However, if you
expose your :meth:`mohawk.Sender.accept_response` call
somewhere publicly over HTTP then you
may need to protect against response replay attacks.
=======
This generally means you don't have to worry about *response* replay attacks.
However, if you
expose your ``acccept_response()`` call somewhere publicly over HTTP then you
may wish to protect against response replay attacks.
>>>>>>> 14f3100... Document all the things
You can do so by constructing a :class:`mohawk.Sender` with
the same ``seen_nonce`` keyword:

.. doctest:: usage

    >>> sender = Sender({'id': 'some-sender',
<<<<<<< HEAD
    ...                  'key': 'a long, complicated secret',
=======
    ...                  'key': 'some complicated SEKRET',
>>>>>>> 14f3100... Document all the things
    ...                  'algorithm': 'sha256'},
    ...                 url,
    ...                 method,
    ...                 content=content,
    ...                 content_type=content_type,
    ...                 seen_nonce=seen_nonce)

.. _`cryptographic nonce`: http://en.wikipedia.org/wiki/Cryptographic_nonce

.. _skipping-content-checks:

Skipping content checks
=======================

<<<<<<< HEAD
In some cases you may not be able to hash request/response content. For
example, the content could be too large. If you run into this, Hawk
might not be the best fit for you but Hawk does allow you to accept
content without a declared hash if you wish.

.. important::

    By allowing content without a declared hash, both the sender and
    receiver are susceptible to content tampering.
=======
In some cases you may not be able to sign request/response content. For example,
the content could be too large to fit in memory. If you run into this, Hawk
might not be the best fit for you but Hawk does allow you to skip content
checks if you wish.

.. important::

    By skipping content checks both the sender and receiver are
    susceptible to content tampering.
>>>>>>> 14f3100... Document all the things

You can send a request without signing the content by passing this keyword
argument to a :class:`mohawk.Sender`:

.. doctest:: usage

    >>> sender = Sender(credentials, url, method, always_hash_content=False)

This says to skip hashing of the ``content`` and ``content_type`` values
<<<<<<< HEAD
if they are both :data:`mohawk.base.EmptyValue`.
if they are both :attr:``mohawk.EmptyValue``.
=======
if they are both ``None``.
>>>>>>> 14f3100... Document all the things

Now you'll get an ``Authorization`` header without a ``hash`` attribute:

.. doctest:: usage

    >>> sender.request_header
    'Hawk mac="...", id="some-sender", ts="...", nonce="..."'

<<<<<<< HEAD
The :class:`mohawk.Receiver` must also be constructed to accept content
without a declared hash using ``accept_untrusted_content=True``:
=======
The :class:`mohawk.Receiver` must also be constructed to
accept unsigned content, like this:
>>>>>>> 14f3100... Document all the things

.. doctest:: usage

    >>> receiver = Receiver(lookup_credentials,
    ...                     sender.request_header,
    ...                     request['url'],
    ...                     request['method'],
<<<<<<< HEAD
    ...                     content=request['content'],
    ...                     content_type=request['headers']['Content-Type'],
    ...                     accept_untrusted_content=True)

This will skip checking the hash of ``content`` and ``content_type`` only if
the ``Authorization`` header omits the ``hash`` attribute. If the ``hash``
attribute is present, it will be checked as normal.

.. _empty-requests:

Empty requests
==============

For requests whose ``content`` (and by extension ``content_type``) is ``None``
or an empty string, it is acceptable for the sender to omit the declared hash,
For requests whose ``content`` (and by extension ``content_type``) is ``None``
or ``''``, it is acceptable for the sender to omit the declared hash,
regardless of the ``accept_untrusted_content`` value provided to the
:class:`mohawk.Receiver`. For example, a ``GET`` request typically has
empty content and some libraries may or may not hash the content.

If the ``hash`` attribute *is* present, a ``None`` value for either
``content`` or ``content_type`` will be coerced to an empty string
prior to hashing.

Generating protected URLs
=========================

Hawk lets you protect a URL with a token derived from a secret key.
After a period of time, access to the URL will expire.
As an example, you could use this to deliver a URL for purchased media,
such a zip file of MP3s. The user could access the URL for a short period
of time but after that, the same URL would not be accessible.

In the Hawk spec, this is referred to as `Single URI Authorization, or bewit`_.

.. _`Single URI Authorization, or bewit`: https://github.com/hueniverse/hawk/#single-uri-authorization

Here's an example of protecting access to this URL with Mohawk:

.. doctest:: usage

    >>> url = 'https://site.org/purchases/music-album.zip'

Let's say you want to allow access for 5 minutes:


.. doctest:: usage

    >>> from mohawk.util import utc_now
    >>> url_expires_at = utc_now() + (60 * 5)

Set up Hawk credentials like in previous examples:

.. doctest:: usage

    >>> credentials = {
    ...     'id': 'some-recipient',
    ...     'key': 'a long, complicated secret',
    ...     'algorithm': 'sha256'
    ... }


Define the resource that you want to protect:

.. doctest:: usage

    >>> from mohawk.base import Resource
    >>> resource = Resource(
    ...     credentials=credentials,
    ...     url=url,
    ...     method='GET',
    ...     nonce='',
    ...     timestamp=url_expires_at,
    ... )

Generate a bewit token:

.. doctest:: usage

    >>> from mohawk.bewit import get_bewit
    >>> bewit = get_bewit(resource)

Add that token as a ``bewit`` query string parameter back to the same URL:

.. doctest:: usage

    >>> protected_url = '{url}?bewit={bewit}'.format(url=url, bewit=bewit)
    >>> protected_url
    'https://site.org/purchases/music-album.zip?bewit=...'

Now you can deliver this bewit protected URL to the recipient.

Serving protected URLs
======================

When handling a request for a bewit protected URL on the server, you can
begin by checking the bewit to make sure it's valid.
If ``True``, the server can respond with access to the resource.
The ``check_bewit`` function returns ``True`` or ``False`` and will also
raise an exception for invalid ``bewit`` values.

.. doctest:: usage

    >>> allowed_recipients = {}
    >>> allowed_recipients['some-recipient'] = credentials
    >>> def lookup_credentials(recipient_id):
    ...     if recipient_id in allowed_recipients:
    ...         # Return a credentials dictionary
    ...         return allowed_recipients[recipient_id]
    ...     else:
    ...         raise LookupError('unknown recipient_id')
    >>> from mohawk.bewit import check_bewit
    >>> check_bewit(protected_url, credential_lookup=lookup_credentials)
    True


.. note::

   Well, that was complicated! At a future time,
   ``get_bewit`` and ``check_bewit`` will be complimented with a higher
   level function that is easier to work with.
   See https://github.com/kumar303/mohawk/issues/17

If the ``hash`` attribute is present and ``accept_untrusted_content`` is
``False``, a ``None`` value for either ``content`` or ``content_type``will
be coerced to ``''`` prior to hashing. This is to account for some dependent
libraries that may provide the empty string even when no content is present
on the request.

=======
    ...                     accept_untrusted_content=True)

>>>>>>> 14f3100... Document all the things
Logging
=======

All internal `logging <http://docs.python.org/2/library/logging.html>`_
channels stem from ``mohawk``. For example, the ``mohawk.receiver``
channel will just contain receiver messages. These channels correspond
to the submodules within mohawk.

<<<<<<< HEAD
To debug :class:`mohawk.exc.MacMismatch` :ref:`exceptions`
=======
To debug :class:`mohawk.exc.MacMismatch` exceptions
>>>>>>> 14f3100... Document all the things
and other authorization errors, set the ``mohawk`` channel to ``DEBUG``.

Going further
=============

Well, hey, that about summarizes the concepts and basic usage of Mohawk.
Check out the :ref:`API` for details.
Also make sure you are familiar with :ref:`security`.

<<<<<<< HEAD
.. _`TLSdate`: http://linux-audit.com/tlsdate-the-secure-alternative-for-ntpd-ntpdate-and-rdate/
=======
>>>>>>> 14f3100... Document all the things
.. _`Hawk`: https://github.com/hueniverse/hawk
.. _`requests`: http://docs.python-requests.org/
