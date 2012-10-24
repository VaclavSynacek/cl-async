---
title: Base system | Documentation
layout: default
---

<a id="base-system"></a>
Base system
===========
This section describes the base functions for cl-async and also the low-level
conditions it uses.

- [start-event-loop](#start-event-loop) _function_
- [exit-event-loop](#exit-event-loop) _function_
- [connection-info](#connection-info) _condition_
- [connection-error](#connection-error) _condition_
  - [conn-errcode](#conn-errcode) _accessor_
  - [conn-errmsg](#conn-errmsg) _accessor_

<a id="start-event-loop"></a>
### start-event-loop
Start the event loop, giving a function that will be run inside the event loop
once started. `start-event-loop` blocks the main thread until the event loop
returns, which doesn't happen until the loop is empty *or*
[exit-event-loop](#exit-event-loop) is called inside the loop.

This function must be called before any other operations in the library are
allowed. If you try to do an async operation without an event loop running, it
will throw an error.

It allows specifying callbacks for the fatal errors in libevent (called when
libevent would normally exit, taking your app with it), logging, and default
application error handling.

{% highlight cl %}
;; definition:
(start-event-loop start-fn &key fatal-cb logger-cb default-event-cb catch-app-errors)

;; example:
(start-event-loop (lambda () (format t "Event loop started.~%")))
{% endhighlight %}

<a id="fatal-cb-definition"></a>
##### fatal-cb definition

{% highlight cl %}
(lambda (errcode) ...)
{% endhighlight %}

<a id="logger-cb-definition"></a>
##### logger-cb definition

{% highlight cl %}
(lambda (loglevel msg) ...)
{% endhighlight %}

`loglevel` corresponds to syslog levels.

<a id="default-event-cb"></a>
##### default-event-cb and catch-app-errors
Please see the [application error handling](#application-error-handling) section
for complete information on these. They correspond 1 to 1 with
[\*default-event-handler\*](#default-event-handler) and
[\*catch-application-errors\*](#catch-application-errors). Setting them when
calling `start-event-loop` not only cuts down on `setf`s you have to do when
starting your evented app, but also uses thread-local versions of the vars,
meaning you can start multiple event loops in multiple threads wiithout using
the same values for each thread.

<a id="exit-event-loop"></a>
### exit-event-loop
Exit the event loop. This will free up all resources internally and close down
the event loop.

Note that this doesn't let queued events process, and is the equivelent of
doing a force close. Unless you really need to do this and return control to
lisp, try to let your event loop exit of "natural causes" (ie, no events left to
process). You can do this by freeing your signal handlers, servers, etc. This
has the added benefit of letting any connected clients finish their requests
(without accepting new ones) without completely cutting them off.

{% highlight cl %}
;; definition
(exit-event-loop)
{% endhighlight %}

<a id="connection-info"></a>
### connection-info
Base connection condition. Signals that "something" happened on a connection.
Meant to be extended.

<a id="connection-error"></a>
### connection-error
_extends [connection-info](#connection-info)_

Base connection error. Signals that "something bad" happened on a connection.

<a id="conn-errcode"></a>
##### conn-errcode
The error code associated with the connection error. This is generally retrieved
from the underlying OS, but sometimes cl-async will generate its own error
conditions, in which case errcode will be -1.

<a id="conn-errmsg"></a>
##### conn-errmsg
Much like `conn-errcode`, this is generally a system message explaining a
connection error. If it is a cl-async generated error, it will have a string
value explaining what happened.
