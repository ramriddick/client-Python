# ReportPortal python client

[![PyPI](https://img.shields.io/pypi/v/reportportal-client.svg?maxAge=2592000)](https://pypi.python.org/pypi/reportportal-client)
[![Build Status](https://travis-ci.org/reportportal/client-Python.svg?branch=master)](https://travis-ci.org/reportportal/client-Python)

Library used only for implementors of custom listeners for ReportPortal


## Already implemented listeners:

- [PyTest Framework](https://github.com/reportportal/agent-python-pytest)
- [Robot Framework](https://github.com/reportportal/agent-Python-RobotFramework)


## Installation

The latest stable version is available on PyPI:

```
pip install reportportal-client
```


## Usage

Main classes are:

- reportportal_client.ReportPortalService
- reportportal_client.ReportPortalServiceAsync

Basic usage example:

```python
import os
import subprocess
import traceback
from mimetypes import guess_type
from time import time

from reportportal_client import ReportPortalServiceAsync


def timestamp():
    return str(int(time() * 1000))


endpoint = "http://10.6.40.6:8080"
project = "default"
# You can get UUID from user profile page in the Report Portal.
token = "1adf271d-505f-44a8-ad71-0afbdf8c83bd"
launch_name = "Test launch"
launch_doc = "Testing logging with attachment."


def my_error_handler(exc_info):
    """
    This callback function will be called by async service client when error occurs.
    Return True if error is not critical and you want to continue work.
    :param exc_info: result of sys.exc_info() -> (type, value, traceback)
    :return:
    """
    print("Error occurred: {}".format(exc_info[1]))
    traceback.print_exception(*exc_info)


service = ReportPortalServiceAsync(endpoint=endpoint, project=project,
                                   token=token, error_handler=my_error_handler)

# Start launch.
launch = service.start_launch(name=launch_name,
                              start_time=timestamp(),
                              description=launch_doc)

# Start test item.
test = service.start_test_item(name="Test Case",
                               description="First Test Case",
                               tags=["Image", "Smoke"],
                               start_time=timestamp(),
                               item_type="STEP")

# Create text log message with INFO level.
service.log(time=timestamp(),
            message="Hello World!",
            level="INFO")

# Create log message with attached text output and WARN level.
service.log(time=timestamp(),
            message="Too high memory usage!",
            level="WARN",
            attachment={
                "name": "free_memory.txt",
                "data": subprocess.check_output("free -h".split()),
                "mime": "text/plain"
            })

# Create log message with binary file, INFO level and custom mimetype.
image = "/tmp/image.png"
with open(image, "rb") as fh:
    attachment = {
        "name": os.path.basename(image),
        "data": fh.read(),
        "mime": guess_type(image)[0] or "application/octet-stream"
    }
    service.log(timestamp(), "Screen shot of issue.", "INFO", attachment)

# Create log message supplying only contents
service.log(
    timestamp(),
    "running processes",
    "INFO",
    attachment=subprocess.check_output("ps aux".split()))

# Finish test item.
service.finish_test_item(end_time=timestamp(), status="PASSED")

# Finish launch.
service.finish_launch(end_time=timestamp())

# Due to async nature of the service we need to call terminate() method which
# ensures all pending requests to server are processed.
# Failure to call terminate() may result in lost data.
service.terminate()
```


# Copyright Notice

Licensed under the [GPLv3](https://www.gnu.org/licenses/quick-guide-gplv3.html)
license (see the LICENSE.txt file).
