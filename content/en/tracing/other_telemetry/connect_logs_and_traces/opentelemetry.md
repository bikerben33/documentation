---
title: Correlating OpenTelemetry Traces and Logs
description: 'Connect your application logs and OpenTelemetry traces to correlate them in Datadog'
code_lang: opentelemetry
type: multi-code-lang
code_lang_weight: 80
aliases:
  - /tracing/connect_logs_and_traces/opentelemetry
further_reading:
- link: "/opentelemetry/otel_tracing/"
  tag: "Documentation"
  text: "Send OpenTelemetry Traces to Datadog"
- link: "https://opentelemetry.io/docs/collector/"
  tag: "External Site"
  text: "Collector documentation"
- link: "https://www.datadoghq.com/blog/opentelemetry-instrumentation/"
  tag: "Blog"
  text: "Datadog's partnership with OpenTelemetry"
- link: '/logs/guide/ease-troubleshooting-with-cross-product-correlation/'
  tag: 'Guide'
  text: 'Ease troubleshooting with cross product correlation.'
---

<div class="alert alert-info">If you are using the latest versions of Datadog tracing libraries, Datadog automatically links OpenTelemetry traces and logs using <code>TraceId</code>. If you're using older versions, follow the steps on this page to manually correlate traces and logs.</div>

Connecting OpenTelemetry language SDK logs and traces within Datadog is similar to connecting [Datadog SDK logs and traces][1], with a few additional steps:

1. OpenTelemetry `TraceId` and `SpanId` properties differ from Datadog conventions. Therefore it's necessary to translate `TraceId` and `SpanId` from their OpenTelemetry formats ([a 128bit unsigned int and 64bit unsigned int represented as a 32-hex-character and 16-hex-character lowercase string, respectively][2]) into their Datadog Formats([a 64bit unsigned int][3]). 

2. Ensure your logs are sent as JSON, because your language level logs must be turned into Datadog attributes for trace-log correlation to work.

See the following examples for language-specific information about how to correlate your OpenTelemetry traces and logs.

{{< tabs >}}
{{% tab "Python" %}}

To manually correlate your traces with your logs, patch the logging module you are using with a processor that translates OpenTelemetry formatted `trace_id` and `span_id` into the Datadog format. The following example uses the [structlog logging library][1]. For other logging libraries, it may be more appropriate to [modify the Datadog SDK examples][2]. You can also find [an example OpenTelemetry instrumented Python application with trace and log correlation][3] in the `trace-examples` GitHub repository.

```python
# ########## injection.py
from opentelemetry import trace

class CustomDatadogLogProcessor(object):
    def __call__(self, logger, method_name, event_dict):
        # An example of adding datadog formatted trace context to logs
        # from: https://github.com/open-telemetry/opentelemetry-python-contrib/blob/b53b9a012f76c4fc883c3c245fddc29142706d0d/exporter/opentelemetry-exporter-datadog/src/opentelemetry/exporter/datadog/propagator.py#L122-L129 
        current_span = trace.get_current_span()
        if not current_span.is_recording():
            return event_dict

        context = current_span.get_span_context() if current_span is not None else None
        if context is not None:
            event_dict["dd.trace_id"] = str(context.trace_id & 0xFFFFFFFFFFFFFFFF)
            event_dict["dd.span_id"] = str(context.span_id)

        return event_dict        
# ##########

# ########## app.py
import .injection
import logging
import structlog
# Add custom formatting to inject datadog formatted trace ids into logs
structlog.configure(
    processors=[
        injection.CustomDatadogLogProcessor(),
        structlog.processors.JSONRenderer(sort_keys=True)
    ],
)

log = structlog.getLogger()

log.info("Example log line with trace correlation info")
```

#### Alternative approach

You can also use unstructured logs with OpenTelemetry SDK logic to correlate logs and traces in your application.

**Note**: This approach uses OpenTelemetry-native trace context fields (not `dd.trace_id` or `dd.span_id`), which can still be correlated in Datadog using remappers. See below for details.

1. Set up OpenTelemetry logging instrumentation in your Python application:

   ```python
   import logging
   from opentelemetry.instrumentation.logging import LoggingInstrumentor
   
   # Initialize the OpenTelemetry logging instrumentation
   LoggingInstrumentor().instrument(set_logging_format=True)
   
   # Create a logger instance
   logger = logging.getLogger(__name__)
   
   # Log a message with automatic trace context injection
   logger.info("This is a log message")
   ```

2. Verify your log format contains the trace context information:

   Your logs should now include trace context information and look similar to:
   ```
   2025-03-25 10:31:52,116 INFO [__main__] [test-logging.py:9] [trace_id=0 span_id=0 resource.service.name= trace_sampled=False] - This is    a log message
   ```

   Or in a real-world scenario:
   ```
   2025-03-20 12:45:10,123 INFO [jobs.scheduler.task_runner.execute] [task_runner.py:123] [trace_id=123abc456def789ghi012jkl345mno67    span_id=89ab01cd23ef45gh resource.service.name=job_scheduler trace_sampled=True] - STARTED JOB TASK. Success
   ```

3. Create a Log Pipeline in Datadog with this Grok Parser Rule:

   ```
   # Define prefix components
   _timestamp %{date("yyyy-MM-dd HH:mm:ss','SSS"):timestamp}
   _level %{word:level}
   _module \[%{notSpace:module}\]
   _file_location \[%{notSpace:file_location}\]
   _trace_info \[trace_id=%{notSpace:trace_id} span_id=%{notSpace:span_id} resource\.service\.name=%{notSpace:service_name} trace_sampled=%   {notSpace:trace_sampled}\]
   
   # Complete rule
   custom_format %{_timestamp} %{_level} %{_module} %{_file_location} %{_trace_info} - %{data:message}
   ```

After setting up the Grok Parser Rule, add the `Trace Id Remapper` and `Span Id Remapper` processors to your pipeline to extract the `trace_id` and `span_id` values, respectively. This configuration allows your logs to appear properly correlated with traces in Datadog.

[1]: https://www.structlog.org/en/stable/standard-library.html
[2]: /tracing/other_telemetry/connect_logs_and_traces/python/#manually-inject-trace-and-span-ids
[3]: https://github.com/DataDog/trace-examples/blob/98626d924f82666de60d6b2d6a65d87eebebdff1/opentelemetry/python-microservice/ddlogging/injection.py#L3
{{% /tab %}}

{{% tab "Node.js" %}}

To manually correlate your traces with your logs, patch the logging module you are using with a processor that translates OpenTelemetry formatted `trace_id` and `span_id` into the Datadog format. The following example uses the [winston logging library][1]. For other logging libraries, it may be more appropriate to [modify the Datadog SDK examples][2]. You can also find [an example OpenTelemetry instrumented Node.js application with trace and log correlation][3] in the `trace-examples` GitHub repository.

```js
// ########## logger.js

// convert to dd with:
// https://github.com/DataDog/dd-trace-js/blob/master/packages/dd-trace/src/id.js
const opentelemetry = require('@opentelemetry/api');
const winston = require('winston')

const tracingFormat = function () {
  return winston.format(info => {
    const span = opentelemetry.trace.getSpan(opentelemetry.context.active());
    if (span) {
      const { spanId, traceId } = span.spanContext();
      const traceIdEnd = traceId.slice(traceId.length / 2);
      info['dd.trace_id'] = BigInt(`0x${traceIdEnd}`).toString();
      info['dd.span_id'] = BigInt(`0x${spanId}`).toString();
    }
    return info;
  })();
}

module.exports = winston.createLogger({
  transports: [new winston.transports.Console],
  format: winston.format.combine(tracingFormat(), winston.format.json())
});

// ##########

// ########## index.js
//
// ...
// initialize your tracer
// ...
// 
const logger = require('./logger') 
//
// use the logger in your application
logger.info("Example log line with trace correlation info")
```



[1]: https://github.com/winstonjs/winston
[2]: /tracing/other_telemetry/connect_logs_and_traces/nodejs/#manually-inject-trace-and-span-ids
[3]: https://github.com/DataDog/trace-examples/blob/98626d924f82666de60d6b2d6a65d87eebebdff1/opentelemetry/node-microservice/logger.js#L86
{{% /tab %}}

{{% tab "Ruby" %}}

To manually correlate your traces with your logs, patch the logging module you are using with a processor that translates OpenTelemetry formatted `trace_id` and `span_id` into the Datadog format. The following example uses the [Ruby Standard Logging Library][1]. For Rails applications or other logging libraries, it may be more appropriate to [modify the Datadog SDK examples][2]. You can also find [an example OpenTelemetry instrumented Ruby application with trace and log correlation][3] in the `trace-examples` GitHub repository.

```ruby
logger = Logger.new(STDOUT)
logger.progname = 'multivac'
original_formatter = Logger::Formatter.new
logger.formatter  = proc do |severity, datetime, progname, msg|
  current_span = OpenTelemetry::Trace.current_span(OpenTelemetry::Context.current).context
  
  dd_trace_id = current_span.trace_id.unpack1('H*')[16, 16].to_i(16).to_s
  dd_span_id = current_span.span_id.unpack1('H*').to_i(16).to_s
  
  if current_span
    "#{{datetime: datetime, progname: progname, severity: severity, msg: msg, 'dd.trace_id': dd_trace_id, 'dd.span_id': dd_span_id}.to_json}\n"
  else
    "#{{datetime: datetime, progname: progname, severity: severity, msg: msg}.to_json}\n"
  end
end

logger.info("Example log line with trace correlation info")
```




[1]: https://ruby-doc.org/stdlib-3.0.0/libdoc/logger/rdoc/index.html
[2]: /tracing/other_telemetry/connect_logs_and_traces/ruby/#manually-inject-trace-and-span-ids
[3]: https://github.com/DataDog/trace-examples/blob/98626d924f82666de60d6b2d6a65d87eebebdff1/opentelemetry/ruby-microservice/app.rb#L21-L35
{{% /tab %}}

{{% tab "Java" %}}

To manually correlate your traces with your logs, first enable the [openTelemetry-java-instrumentation Logger MDC Instrumentation][1]. Then, patch the logging module you are using with a processor that translates OpenTelemetry formatted `trace_id` and `span_id` into the Datadog format. The following example uses [Spring Boot and Logback][2]. For other logging libraries, it may be more appropriate to [modify the Datadog SDK examples][3]. 

```java
String traceIdValue = Span.current().getSpanContext().getTraceId();
String traceIdHexString = traceIdValue.substring(traceIdValue.length() - 16 );
long datadogTraceId = Long.parseUnsignedLong(traceIdHexString, 16);
String datadogTraceIdString = Long.toUnsignedString(datadogTraceId);

String spanIdHexString = Span.current().getSpanContext().getSpanId();
long datadogSpanId = Long.parseUnsignedLong(spanIdHexString, 16);
String datadogSpanIdString = Long.toUnsignedString(datadogSpanId);

logging.pattern.console = %d{yyyy-MM-dd HH:mm:ss} - %logger{36} - %msg dd.trace_id=%X{datadogTraceIdString} dd.span_id=%X{datadogSpanIdString} %n
```

See [Java Log Collection][4] on how to send your Java logs to Datadog.

[1]: https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/main/docs/logger-mdc-instrumentation.md
[2]: https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-logging.html
[3]: /tracing/other_telemetry/connect_logs_and_traces/java/?tab=log4j2#manually-inject-trace-and-span-ids
[4]: /logs/log_collection/java/?tab=logback
{{% /tab %}}

{{% tab "PHP" %}}

For trace and log correlation in PHP, modify the [Datadog SDK PHP examples][1] to include the additional steps discussed above.

[Contact Datadog support][2] with any questions.




[1]: /tracing/other_telemetry/connect_logs_and_traces/php/
[2]: /help/
{{% /tab %}}

{{% tab "Go" %}}

To manually correlate your traces with your logs, patch the logging module you are using with a function that translates OpenTelemetry formatted `trace_id` and `span_id` into the Datadog format. The following example uses the [logrus Library][1].

```go
package main

import (
	"context"
	log "github.com/sirupsen/logrus"
	"go.opentelemetry.io/otel"
	"strconv"
)

func main() {
	ctx := context.Background()
	tracer := otel.Tracer("example/main")
	ctx, span := tracer.Start(ctx, "example")
	defer span.End()

	log.SetFormatter(&log.JSONFormatter{})

	standardFields := log.Fields{
		"dd.trace_id": convertTraceID(span.SpanContext().TraceID().String()),
		"dd.span_id":  convertTraceID(span.SpanContext().SpanID().String()),
		"dd.service":  "serviceName",
		"dd.env":      "serviceEnv",
		"dd.version":  "serviceVersion",
	}

	log.WithFields(standardFields).WithContext(ctx).Info("hello world")
}

func convertTraceID(id string) string {
	if len(id) < 16 {
		return ""
	}
	if len(id) > 16 {
		id = id[16:]
	}
	intValue, err := strconv.ParseUint(id, 16, 64)
	if err != nil {
		return ""
	}
	return strconv.FormatUint(intValue, 10)
}


```

[Contact Datadog support][2] with any questions.



[1]: https://github.com/sirupsen/logrus
[2]: /help/
{{% /tab %}}

{{% tab ".NET" %}}

To manually correlate traces with logs, convert the OpenTelemetry `TraceId` and `SpanId` into the format used by Datadog. Add those IDs to your logs under the `dd.trace_id` and `dd.span_id` attributes. The following example uses the [Serilog library][1], and shows how to convert the OpenTelemetry (`System.DiagnosticSource.Activity`) trace and span IDs into Datadog's required format:

```csharp
var stringTraceId = Activity.Current.TraceId.ToString();
var stringSpanId = Activity.Current.SpanId.ToString();

var ddTraceId = Convert.ToUInt64(stringTraceId.Substring(16), 16).ToString();
var ddSpanId = Convert.ToUInt64(stringSpanId, 16).ToString();

using (LogContext.PushProperty("dd.trace_id", ddTraceId))
using (LogContext.PushProperty("dd.span_id", ddSpanId))
{
    Serilog.Log.Logger.Information("Example log line with trace correlation info");
}
```


[1]: https://serilog.net/
{{% /tab %}}

{{< /tabs >}}

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /tracing/other_telemetry/connect_logs_and_traces/
[2]: https://github.com/open-telemetry/opentelemetry-specification/blob/eeef21259a12d61100804eff2e12ba06523821c3/specification/trace/api.md#retrieving-the-traceid-and-spanid
[3]: /api/latest/tracing/#send-traces
