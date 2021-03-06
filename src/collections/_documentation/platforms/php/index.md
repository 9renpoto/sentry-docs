---
title: PHP
sidebar_order: 3
---

{% include learn-sdk.md platform="php" %}

The [Sentry PHP SDK](https://packagist.org/packages/sentry/sentry) provides
support for PHP 7.1 or later.

This documentation goes over some PHP specific things such as integrations to
frameworks.

## Integrations

*Integrations* extend the functionality of the SDK for some common frameworks and
libraries.  Similar to plugins, they extend the functionality of the Sentry SDK.  
Integrations are configured by a call to `init()` method.

## PHP Specific Options

PHP provides some additional options, all options can be passed into `init()`.

#### send_attempts

The number of attempts that should be made to send an event before erroring
and dropping it from the queue.
By default this option is set to `6`.

#### project_root

The root of the project source code. As Sentry is able to distinguish project
files from third-party ones (e.g. vendors), this option can be configured to
mark the directory containing all the source code of the application.

For example, assuming that the directory structure shown below exists, marking
the project root as `project-folder/src/` means that every file inside that
directory that is part of a stack trace frame will be marked as "application
code".

```

    project-folder/
    ├── vendor/
        ├── foo/
    ├── src/
        ├── bar/ <-- these are going to be marked as application files

``` 

#### enable_compression

If this option is enabled, `gzip` compression will be enabled. Default is `true`.

#### context_lines

This option sets the number of lines of code context to capture. If `null` is
set as the value, no source code lines will be added to each stack trace frame.
By default this option is set to `3`.

#### excluded_exceptions

Sometimes you may want to skip capturing certain exceptions. This option sets
the FQCN of the classes of the exceptions that you don't want to capture. The
check is done using the `instanceof` operator against each item of the array
and if at least one of them passes the event will be discarded.

#### excluded_app_paths

This option configures the list of paths to exclude from the `app_path` detection.

#### prefixes

This option sets the list of prefixes which should be stripped from the filenames
to create relative paths.

## Transport

Transports are the classes in Sentry PHP that are responsible for communicating
with a service in order to deliver an event. There are several types of transports
available out-of-the-box, all of which implement the `TransportInterface`
interface;

- The `NullTransport` which is used in case you do not define a `DSN` in the options. It will not send events.
- The `HttpTransport` which is the default and will be used when the server
  is set in the client configuration.
- The `SpoolTransport` which can be used to defer the sending of events (e.g.
  by putting them into a queue).
  
The examples below pretty much replace the `init()` call.  
Please also keep in mind that once a Client is initialized with a Transport it cannot be
changed.

### NullTransport

Although not so common there could be cases in which you don't want to send
events at all. The `NullTransport` transport does this: it simply ignores
the events, but report them as sent.

```php
use Sentry\ClientBuilder;
use Sentry\Transport\NullTransport;
use Sentry\State\Hub;

$transport = new NullTransport();
$builder = ClientBuilder::create(['dsn' => '___PUBLIC_DSN___']);
$builder->setTransport($transport);

Hub::getCurrent()->bindClient($builder->getClient());
```

{% include components/alert.html
    title="Note"
    content="`Hub::getCurrent()->bindClient($builder->getClient());` is required to make the global function calls to use your new `Client` instead."
    level="warning"
%}

### HttpTransport

The `HttpTransport` sends events over the HTTP protocol using [Httplug](http://httplug.io/).
The best adapter available is automatically selected when creating a client instance
through the client builder, but you can override it using the appropriate methods.

```php
use Sentry\ClientBuilder;
use Sentry\Transport\HttpTransport;
use Sentry\State\Hub;

$options = ['dsn' => '___PUBLIC_DSN___'];

$transport = new HttpTransport($options, HttpAsyncClientDiscovery::find(), MessageFactoryDiscovery::find());

$builder = ClientBuilder::create($options);
$builder->setTransport($transport);

Hub::getCurrent()->bindClient($builder->getClient());
```

{% include components/alert.html
    title="Note"
    content="`Hub::getCurrent()->bindClient($builder->getClient());` is required to make the global function calls to use your new `Client` instead."
    level="warning"
%}

### SpoolTransport

The default behavior is to send events immediately. You may, however, want to
avoid waiting for the communication to the Sentry server that could be slow
or unreliable. Avoid this by choosing the `SpoolTransport` which
stores the events in a queue so that another process can read it and take
care of sending them. Currently only spooling to memory is supported.


```php
use Sentry\ClientBuilder;
use Sentry\Transport\SpoolTransport;
use Sentry\Transport\HttpTransport;
use Sentry\State\Hub;

$options = ['dsn' => '___PUBLIC_DSN___'];

$spool = new MemorySpool();
$transport = new SpoolTransport($spool);
$httpTransport = new HttpTransport($options, HttpAsyncClientDiscovery::find(), MessageFactoryDiscovery::find());

$builder = ClientBuilder::create($options);
$builder->setTransport($transport);

Hub::getCurrent()->bindClient($builder->getClient());

// When the spool queue is flushed the events are sent using the transport
// passed as parameter of the flushQueue method.
  
$spool->flushQueue($httpTransport);
```

{% include components/alert.html
    title="Note"
    content="`Hub::getCurrent()->bindClient($builder->getClient());` is required to make the global function calls to use your new `Client` instead."
    level="warning"
%}
