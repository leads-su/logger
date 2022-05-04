# Logging Package for Go Lang
This package provides ability to log events.

This package also supports log rotation. This functionality will be discussed below.

## General Information
This package allows you to produce logs for both: files and console.

Default output for console is `Text`, for files it is however `JSON`.

This provides ability for better log readability while in console and automatic log collection by systems which support JSON parsing.

### Example of Console output
```sh
WARN[2021-07-12T19:41:26+03:00] Warn Message architecture=amd64 commit=1234567890 runtime=go1.16.3 source="cmd:start" version=0.0.0
```
### Example of JSON output
```json
{
  "architecture": "amd64",
  "commit": "1234567890",
  "level": "warning",
  "msg": "Warn Message",
  "runtime": "go1.16.3",
  "source": "cmd:start",
  "time": "2021-07-12T19:34:48+03:00",
  "version": "0.0.0"
}
```

### Default Log Fields
Logger by default adds several fields to improve the "quality" of produced information, these fields are
- `version` - version of binary
- `commit` - commit used to build binary
- `source` - this feature will be explained later in this README
- `architecture` - architecture on which this binary is running (or with which this binary was built)
- `runtime` - version of GoLang used to build this binary

## Supported Logging Methods

### Trace
Designates finer-grained informational events than the Debug.
```go
logger.Trace(source, message string)
```

### Debug
Usually only enabled when debugging. Very verbose logging.
```go
logger.Debug(source, message string)
```

### Info
General operational entries about what's going on inside the application.
```go
logger.Info(source, message string)
```

### Warn
Non-critical entries that deserve eyes.
```go
logger.Warn(source, message string)
```

### Error
Used for errors that should definitely be noted. Commonly used for hooks to send errors to an error tracking service.
```go
logger.Error(source, message string)
```

### Fatal
Logs and then calls `os.Exit(1)`. It will exit even if the logging level is set to Panic.
```go
logger.Fatal(source, message string)
```

### Panic
Logs and then calls panic with the message passed to Debug, Info, etc...
```go
logger.Panic(source, message string)
```

### Formatting
Each log level comes with the supporting method which allows you to perform value substitution inside the log message.

In order to use formatting, you have to append `f` to the desired log level.

**Signature:**
```go
logger.Panicf(source, message string, args ...interface{})
```

**Example:**
```go
logger.Panicf("database:connection", "failed to connect to database at %s:%d", "localhost", 3306)
```

## Log Rotation
You can create at most two log rotators, one for `info` logs and one for `error` logs.

**There are 3 possible ways to create rotators:**
1. Create rotator for `info` logs - ```logger.RegisterOutputRotator(infoOutput)```
```go
logger.RegisterOutputRotator(logger.RotatorFileConfig{
	FileName:   "/var/log/app/info.log",
	MaxSize:    1,
	MaxBackups: 3,
	MaxAge:     1,
	Level:      logrus.TraceLevel,
	Formatter:  &logrus.JSONFormatter{
		TimestampFormat: time.RFC3339,
	},
	Compress:   false,
})
```
2. Create rotator for `error` logs - ```logger.RegisterErrorRotator(errorOutput)```
```go
logger.RegisterErrorRotator(logger.RotatorFileConfig{
	FileName:   "/var/log/app/error.log",
	MaxSize:    1,
	MaxBackups: 3,
	MaxAge:     1,
	Level:      logrus.TraceLevel,
	Formatter:  &logrus.JSONFormatter{
		TimestampFormat: time.RFC3339,
	},
	Compress:   false,
})
```
3. Create rotators for `info` and `error` logs - ```logger.RegisterRotators(infoOutput, errorOutput)```
```go
logger.RegisterRotators(logger.RotatorFileConfig{
	FileName:   "/var/log/app/info.log",
	MaxSize:    1,
	MaxBackups: 3,
	MaxAge:     1,
	Level:      logrus.TraceLevel,
	Formatter:  &logrus.JSONFormatter{
		TimestampFormat: time.RFC3339,
	},
	Compress:   false,
}, logger.RotatorFileConfig{
	FileName:   "/var/log/app/error.log",
	MaxSize:    1,
	MaxBackups: 3,
	MaxAge:     1,
	Level:      logrus.TraceLevel,
	Formatter:  &logrus.JSONFormatter{
		TimestampFormat: time.RFC3339,
	},
	Compress:   false,
})
```

Configuration is provided in form of ```logger.RotatorFileConfig``` which has the following signature
```go
type RotatorFileConfig struct {
	// FileName defines output file name
	FileName			string
	// MaxSize defines maximum log size in megabytes
	MaxSize				uint
	// MaxBackups defines maximum number of backups for one file
	MaxBackups			uint
	// MaxAge defines maximum age of log in days
	MaxAge				uint
	// Level defines log level
	Level				logrusPackage.Level
	// Formatter defines formatter used by rotator
	Formatter			logrusPackage.Formatter
	// Compress defines whether logs should be compressed
	Compress			bool
}
```