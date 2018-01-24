[![PyPI](https://img.shields.io/pypi/v/pyexos.svg)](https://pypi.python.org/pypi/pyEXOS)
[![Build Status](https://travis-ci.org/LINXNet/pyexos.svg?branch=master)](https://travis-ci.org/LINXNet/pyexos)
[![Coverage Status](https://coveralls.io/repos/github/LINXNet/pyexos/badge.svg?branch=master)](https://coveralls.io/github/LINXNet/pyexos?branch=master)


# pyEXOS

Python library to manage Extreme Networks devices running EXOS

## Requirements

### Python
* netmiko >= 1.4.2

### Extreme Networks
* EXOS version == 16.2

## Install

### Install via pip
```bash
pip install pyEXOS
```

### Upgrade via pip
```bash
pip install --upgrade pyEXOS
```

## Documentation

### Connect to remote device via SSH
```python
>>> from pyEXOS import EXOS
>>> device = EXOS(hostname='192.168.1.222', username='admin', password='admin', port=22, timeout=10)
>>> device.open()
```

### Connect to remote device via Telnet
```python
>>> from pyEXOS import EXOS
>>> device = EXOS(hostname='192.168.1.222', username='admin', password='admin', protocol='telnet', port=23, timeout=10)
>>> device.open()
```

### Load and later discard config
```python
>>> device.load_candidate_config(config='configure snmp sysName test-string')
>>> print device.candidate_config
>>> ['configure snmp sysName test-string']
>>> device.discard_config()
>>> print device.candidate_config
>>> None
```

### Retrieve running config from the device
```python
>>> device.get_running_config()
>>> print device.running_config
>>> [u'...']
```

###  Load, diff merge, commit merge config and later roll back to previous state
```python
>>> device.load_candidate_config(filename='config_merge.txt')
>>> diff = device.compare_merge_config()
>>> print diff
>>> --- running_config.conf
+++ candidate_config.conf
@@ -22,0 +23 @@
+configure snmp sysName test-merge-file
>>> device.commit_config()
```

###  Load, diff replace, commit replace config and later roll back to previous state
```python
>>> device.load_candidate_config(filename='config_replace.txt')
>>> diff = device.compare_replace_config()
>>> print diff
...
>>> device.commit_replace_config()
>>> device.rollback()
```

### Close the connection
```python
>>> device.close()
```

### Logging
Logging is facilitated though the python logging module. Once you initilize a logger in your main program,
pyexos will emit its messages accordingly.
```python
>>> import logging
>>> import sys
>>> logging.basicConfig(stream=sys.stdout, level=logging.INFO)
```


## Caveats

Care needs to be taken with the ```compare_replace_config``` method.
Extreme devices natively don't offer a config replace operation.
Based on the diff with the to-be-applied config, pyEXOS will generate a set of commands to remove already existing config lines one by one.
To generate the commands needed for removal, it uses a list of known commands embedded in the module.
If your config holds a command not known to the module, it will ignore the removal, issue a warning, but the command will remain on the device after the replace operation.
The module will have to be updated and the necessary line added to the ```_generate_commands``` function, to be able to remove this command during the replace operation.


## License

Copyright 2018 LINX

Licensed under the Apache License, Version 2.0: http://www.apache.org/licenses/LICENSE-2.0
