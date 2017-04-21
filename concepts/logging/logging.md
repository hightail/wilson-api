# Wilson Logging

Wilson provides logging functions that are decorated onto the global "wilson" instance. These functions can be
used throughout a wilson application to print logs to the developer console. The interface of these functions is
synonymous with a subset of functions available on the actual console object.

From the code, these functions can be accessed on either the global "wilson" instance or by injecting the "WilsonLogger"
service into any component controller, behavior directive, service or filter declaration.

> The recommended usage is to reference it directly from **wilson.log**

E.G.
```js
function someHandler() {
  try {
    // do some work
    wilson.log.info('Work Completed!');
  } catch(e) {
    wilson.log.error('Work Failed', e);
  }
}
```

Logging in wilson can be controlled by calling the [setLevel()]() function. In production mode, logging is off by
default, this means that all calls to logging functions are no-operation. When running in development mode, the logLevel
is set to **ALL** by default, meaning all messages will be output to the console. Wilson log levels follow the same pattern
as [log4j log levels](https://www.tutorialspoint.com/log4j/log4j_logging_levels.htm)

#### Supported Log Levels:

| Level         | Description                     |
| ----------    | ------------------------------- |
| **ALL**       | All messages                    |
| **TRACE**     | Fine-grained detailed messages  |
| **DEBUG**     | Detailed messages               |
| **INFO**      | Informational messages          |
| **WARN**      | Potential problem messages      |
| **ERROR**     | Recoverable error messages      |
| **FATAL**     | Unrecoverable error messages    |


# Logging Functions

* [trace](#trace)
* [debug](#debug)
* [info](#info)
* [warn](#warn)
* [error](#error)
* [fatal](#fatal)


## <a name="trace"></a>trace(message, ...data)


## <a name="debug"></a>debug(message, ...data)


## <a name="info"></a>info(message, ...data)


## <a name="warn"></a>warn(message, ...data)


## <a name="error"></a>error(message, ...data)


## <a name="fatal"></a>fatal(message, ...data)
