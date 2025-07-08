---
title: "Loggers Loggers Loggers"
description: ""
pubDate: "July 7 2025"
heroImage: "/low_steps.png"
tags: ["dev-peripheral"]
---
The amount of time I spent tweaking various logging-tangencial issues in python is obscene. Good thing I came across this [blog](https://www.dash0.com/guides/logging-in-python) while randomly scrolling through StackOverFlow at work, there's still hope in humanity after all. The following is a brief rundown of the key ideas from the blog, if you have the time, please go ahead and read the original post instead.

### Red Flags
```python
import logging
logging.info("Something finished.")
```
If you see logging like the above in production-level systems at your company, run for your life. Never, never ever log at the root level. Loggers are inherently hierachical objects that inherent settings from their parents, and so one should always use something like
```python
import logging
logger = logging.getLogger(__name__)
```
This helps you take advantage of the natural hierachichal file structure. For example, the logging config in `services.pdater.py` will follow from `services`, which ultimately follows from the root logger.

### Where Are You Going?
**Handlers** are responsible for deciding where the logs are going.
* `StreamHandler` sends logs to a stream (`sys.stdout`, `sys.stderr`) - should be the default config - an application should not concern itself with log file routing or storage, it should allow the execution environment take over. Good for containerized applications like Docker or Kubernetes.
* `FileHandler` used when you manage your own servers or virtual envs. To avoid unbounded log file growth, you can use rotation schemes such as `RotatingFileHandler` or `TimedRotatingFileHandler`
* `QueueHandler` and `QueueListner` are for high-performance, asynchronous, non-blocking logging. It works by attaching a `QueueHandler` to your loggers. Instead of processing the log record immediately, it quickly place the record onto a `queue.Queue`, which is a fast, in-memory operation. We then run a **separate** `QueueListener` thread that continuously monitors the queue for new log records. When it finds one, it removes it from the queue and passes it to one or more downstream handlers for actual handling.
```python
import logging
import logging.handlers
import sys
from queue import Queue

log_queue = Queue(-1) # infinite size queue

# define downstream handler for queue listener
stream_handler = logging.StreamHander(sys.stdout)

# create listener for queue
listener = logging.handlers.QueueListener(log_queue, stream_handler)
listener.start()

queue_handler = logging.handlers.QueueHandler(log_queue)

# configure the logger to use the queue handler alone
logger = logging.getLogger(__name__)
logger.addHandler(queue_handler)

listener.stop()
```
Note that handlers themselves can also filter out certain levels of output by `handler.setLevel(logging.INFO)` etc. This will be applied on top of any logger-level filtering that was already in place.

### Log Filters
```python
import logging

class NoHealthChecksFilter(logging.Filter):
    def filter(self, record: logging.LogRecord) -> bool:
        return "/health" not in record.getMessage()

logger = logging.getLogger("app")
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
handler.addFilter(NoHealthChecksFilter())
logger.addHandler(handler)

logger.info("GET /api request succeeded") # allowed
logger.info("GET /health request succeeded") # dropped
```