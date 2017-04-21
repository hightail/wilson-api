# Wilson Logging

Wilson provides logging functions that are decorated onto the global "wilson" instance. These functions can be
used throughout a wilson application to print logs to the developer console. The interface of these functions is
mostly synonymous with a subset of functions available on the actual console object.

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

Logging in wilson can be controlled by calling the [setLevel()](#setLevel) function. In production mode, logging is off by
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

* [setLevel](#setLevel)
* [trace](#trace)
* [debug](#debug)
* [info](#info)
* [warn](#warn)
* [error](#error)
* [fatal](#fatal)


## <a name="setLevel"></a>setLevel(level)

Sets the current log level of the wilson logger. Accepts a string value representing a valid log level (see above). By default
the log level is off, meaning no messages will print. Setting the log level to a specific value will allow logs to be written
to the console. If a null or unsupported value is provided to this method, the current level will be cleared and logging will
be turned off.

```typescript
function setLevel(level?: string): void;
```
Usage Example:
```js
wilson.log.info('This will not print');         // --> Does nothing
 
wilson.log.setLevel('INFO');
 
wilson.log.info('This info will print');        // --> Prints info message to console
 
wilson.log.error('This error will print');      // --> Prints error message to console
 
wilson.log.debug('This debug will not print');  // --> Does nothing as the logLevel is INFO and ignores debug messages
```

## <a name="trace"></a>trace(message, ...data)

Logs a detailed message and stack trace.

```typescript
function trace(string: message, ...data: any[]): void;
```

## <a name="debug"></a>debug(message, ...data)

Logs a detailed debugging message.

```typescript
function debug(string: message, ...data: any[]): void;
```

## <a name="info"></a>info(message, ...data)

Logs an informational message.

```typescript
function info(string: message, ...data: any[]): void;
```

## <a name="warn"></a>warn(message, ...data)

Logs a warning message. Used to notate potential problems or uncommon scenarios.

```typescript
function warn(string: message, ...data: any[]): void;
```

## <a name="error"></a>error(message, ...data)

Logs an error message. Used to notate recoverable but unexpected error cases.

```typescript
function error(string: message, ...data: any[]): void;
```

## <a name="fatal"></a>fatal(message, ...data)

Logs a fatal error message. Used to notate an unrecoverable error, potentially preventing the application from
continuing to function properly.

```typescript
function fatal(string: message, ...data: any[]): void;
```