# Graylog Hook for [Logrus](https://github.com/Sirupsen/logrus) <img src="http://i.imgur.com/hTeVwmJ.png" width="40" height="40" alt=":walrus:" class="emoji" title=":walrus:" />&nbsp;[![Build Status](https://travis-ci.org/gemnasium/logrus-graylog-hook.svg?branch=master)](https://travis-ci.org/gemnasium/logrus-graylog-hook)&nbsp;[![godoc reference](https://godoc.org/github.com/gemnasium/logrus-graylog-hook?status.png)](https://godoc.org/gopkg.in/gemnasium/logrus-graylog-hook.v1)

Use this hook to send your logs to [Graylog](http://graylog2.org) server over UDP.
The hook is non-blocking: even if UDP is used to send messages, the extra work
should not block the logging function.

All logrus fields will be sent as additional fields on Graylog.

## Usage

The hook must be configured with:

* A Graylog GELF UDP address (a "ip:port" string).
* A facility
* an optional hash with extra global fields. These fields will be included in all messages sent to Graylog

```go
import (
    "log/syslog"
    "github.com/Sirupsen/logrus"
    "gopkg.in/gemnasium/logrus-graylog-hook.v1"
    )

func main() {
    log := logrus.New()
    hook := graylog.NewGraylogHook("<graylog_ip>:<graylog_port>", "some_facility", map[string]interface{}{"this": "is logged every time"})
    log.Hooks.Add(hook)
    log.Info("some logging message")
}
```

### Asynchronous logger

```go
import (
    "log/syslog"
    "github.com/Sirupsen/logrus"
    "gopkg.in/gemnasium/logrus-graylog-hook.v1"
    )

func main() {
    log := logrus.New()
    hook := graylog.NewAsyncGraylogHook("<graylog_ip>:<graylog_port>", "some_facility", map[string]interface{}{"this": "is logged every time"})
    defer hook.Flush()
    log.Hooks.Add(hook)
    log.Info("some logging message")
}
```

### Disable standard logging

For some reason, you may want to disable logging on stdout, and keep only the messages in Graylog (ie: a webserver inside a docker container).
You can redirect `stdout` to `/dev/null`, or just not log anything by creating a `NullFormatter` implementing `logrus.Formatter` interface:

```go
type NullFormatter struct {
}

// Don't spend time formatting logs
func (NullFormatter) Format(e *log.Entry) ([]byte, error) {
    return []byte{}, nil
    }
}
```

And set this formatter as the new logging formatter:

```go
log.Infof("Log messages are now sent to Graylog (udp://%s)", graylogAddr) // Give a hint why logs are empty
log.Hooks.Add(graylog.NewGraylogHook(graylogAddr, "api", map[string]interface{}{})) // set graylogAddr accordingly
log.SetFormatter(new(NullFormatter)) // Don't send logs to stdout
```
