<p align="center">
    <a href="https://www.python.org/psf-landing/" target="_blank">
        <img src="https://www.python.org/static/community_logos/python-logo.png" height="60px">
    </a>
    <h1 align="center">Python Stress Tool</h1>
    <br>
</p>

Stress test tool with statistical TPS reports based on Worker Dispatcher in Python

[![PyPI](https://img.shields.io/pypi/v/stress-tool)](https://pypi.org/project/stress-tool/)
![](https://img.shields.io/pypi/implementation/stress-tool)



Features
--------

- Based on ***[Worker Dispatcher](https://github.com/yidas/python-worker-dispatcher)** to managed workers*

- ***Statistical TPS Report** in Excel sheets*

- ***Customized Config** for the report*  


---

OUTLINE
-------

- [Demonstration](#demonstration)
- [Introduction](#introduction)
- [Installation](#installation)
- [Usage](#usage)
    - [generate_report()](#generate_report)

---

DEMONSTRATION
-------------

Just write your own callback functions based on the [Worker Dispatcher](https://github.com/yidas/python-worker-dispatcher) library, then run it and generate the report file:

```python
import stress_test

def each_task(id: int, config, task, log):
    response = requests.get(config['my_endpoint'] + task)
    return response

def main():
    results = stress_test.start({
        'task': {
            'list': ['ORD_AH001', 'ORD_KL502', '...' , 'ORD_GR393'],
            'callback': each_task,
            'config': {
                'my_endpoint': 'https://your.name/order-handler/'
            },
        }
    })
    # Generate the TPS report if the stress test completes successfully.
    if results != False:
        file_path = stress_test.generate_report(file_path='./tps-report.xlsx')
        print("Report has been successfully generated at {}".format(file_path))

if __name__ == '__main__':
    main()
```

<img src="https://github.com/yidas/python-stress-tool/blob/main/img/demonstration_excel.png?raw=true" />

---

INTRODUCTION
------------

This tool generates professional TPS report based on the execution result from the [Worker Dispatcher](https://github.com/yidas/python-worker-dispatcher) library.

Dependencies:
- [worker-dispatcher](https://github.com/yidas/python-worker-dispatcher)
- openpyxl

---

INSTALLATION
------------

To install the current release:

```shell
$ pip install stress-tool
```

Import it in your Pythone code:

```python
import stress_test
```

---

USAGE
-----

By calling the `start()` method with the configuration parameters, the package will invoke [Worker Dispatcher](https://github.com/yidas/python-worker-dispatcher) to dispatch tasks, managing threading or processing based on the provided settings. Once the tasks are completed, `generate_report()` can be called to produce a TPS report based on the result of [Worker Dispatcher](https://github.com/yidas/python-worker-dispatcher).

### generate_report()

An example configuration setting with all options is as follows:

```python
def generate_report(config: dict={}, worker_dispatcher: object=None, file_path: str='./tps-report.xlsx', display_intervals: bool=True, interval: float=1):
```

#### config

|Option            |Type     |Deafult      |Description|
|:--               |:--      |:--          |:--        |
|raw_logs.fields   |dict     |None         |Customized field setting for the `Raw Logs` sheet. <BR>Key is field name, the value can be two types:<BR> - String: Use the provided value as the log key name from Worker Dispatcher to retrieve the value. <BR> - lambda function: Retrieve the return value from the lambda function.|

#### Sample config

```python
import stress_tool
import requests

# task.callback function
def task(id: int, config, task, log):
    try:
        response = log['response'] = requests.get('https://your.name/path/')
        try:
            api_return_code = log['api_return_code'] = response.json().get('returnCode')
            return True if api_return_code == "0000" else False
        except Exception as e:
            return False
    except requests.exceptions.ConnectionError:
        log['error'] = 'ConnectionError'
    return False

# Start stress test
results = stress_tool.start({
    # 'debug': True,
    'task': {
        'list': 60,
        'callback': task,
    },
})

# Generate the report
file_path = stress_test.generate_report(config={
    'raw_logs': {
        'fields': {
            'Customized Field - HTTP code': lambda log: log.get('response').status_code,
            'Customized Field - API Return code': 'api_return_code',
            'Customized Field - Response Body': lambda log: log.get('response').text,
        }
    },
})

```

#### display_intervals

Indicates whether to generate `Intervals` sheet.

#### interval

Based on `Intervals` sheet, specifies the number of seconds for each split.




