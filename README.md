# smbus2
A drop-in replacement for smbus-cffi/smbus-python in pure Python

[![Build Status](https://github.com/kplindegaard/smbus2/actions/workflows/python-build-test.yml/badge.svg?branch=master)](https://github.com/kplindegaard/smbus2/actions/workflows/python-build-test.yml)
[![Documentation Status](https://readthedocs.org/projects/smbus2/badge/?version=latest)](http://smbus2.readthedocs.io/en/latest/?badge=latest)
![CodeQL](https://github.com/kplindegaard/smbus2/actions/workflows/codeql-analysis.yml/badge.svg?branch=master)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=kplindegaard_smbus2&metric=alert_status)](https://sonarcloud.io/dashboard?id=kplindegaard_smbus2)

![Python Verions](https://img.shields.io/pypi/pyversions/smbus2.svg)
[![PyPi Version](https://img.shields.io/pypi/v/smbus2.svg)](https://pypi.org/project/smbus2/)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/smbus2)](https://pypi.org/project/smbus2/)

# Introduction

smbus2 is (yet another) pure Python implementation of the [python-smbus](http://www.lm-sensors.org/browser/i2c-tools/trunk/py-smbus/) package.

It was designed from the ground up with two goals in mind:

1. It should be a drop-in replacement of smbus. The syntax shall be the same.
2. Use the inherent i2c structs and unions to a greater extent than other pure Python implementations like [pysmbus](https://github.com/bjornt/pysmbus) does. By doing so, it will be more feature complete and easier to extend.

Currently supported features are:

* Get i2c capabilities (I2C_FUNCS)
* SMBus Packet Error Checking (PEC) support
* read_byte
* write_byte
* read_byte_data
* write_byte_data
* read_word_data
* write_word_data
* read_i2c_block_data
* write_i2c_block_data
* write_quick
* process_call
* read_block_data
* write_block_data
* block_process_call
* i2c_rdwr - *combined write/read transactions with repeated start*

It is developed on Python 2.7 but works without any modifications in Python 3.X too.

More information about updates and general changes are recorded in the [change log](https://github.com/kplindegaard/smbus2/blob/master/CHANGELOG.md).

# SMBus code examples

smbus2 installs next to smbus as the package, so it's not really a 100% replacement. You must change the module name.

## Example 1a: Read a byte

```python
from smbus2 import SMBus

# Open i2c bus 1 and read one byte from address 80, offset 0
bus = SMBus(1)
b = bus.read_byte_data(80, 0)
print(b)
bus.close()
```

## Example 1b: Read a byte using 'with'

This is the very same example but safer to use since the smbus will be closed automatically when exiting the with block.

```python
from smbus2 import SMBus

with SMBus(1) as bus:
    b = bus.read_byte_data(80, 0)
    print(b)
```

## Example 1c: Read a byte with PEC enabled

Same example with Packet Error Checking enabled.

```python
from smbus2 import SMBus

with SMBus(1) as bus:
    bus.pec = 1  # Enable PEC
    b = bus.read_byte_data(80, 0)
    print(b)
```

## Example 2: Read a block of data

You can read up to 32 bytes at once.

```python
from smbus2 import SMBus

with SMBus(1) as bus:
    # Read a block of 16 bytes from address 80, offset 0
    block = bus.read_i2c_block_data(80, 0, 16)
    # Returned value is a list of 16 bytes
    print(block)
```

## Example 3: Write a byte

```python
from smbus2 import SMBus

with SMBus(1) as bus:
    # Write a byte to address 80, offset 0
    data = 45
    bus.write_byte_data(80, 0, data)
```

## Example 4: Write a block of data

It is possible to write 32 bytes at the time, but I have found that error-prone. Write less and add a delay in between if you run into trouble.

```python
from smbus2 import SMBus

with SMBus(1) as bus:
    # Write a block of 8 bytes to address 80 from offset 0
    data = [1, 2, 3, 4, 5, 6, 7, 8]
    bus.write_i2c_block_data(80, 0, data)
```

# I2C

Starting with v0.2, the smbus2 library also has support for combined read and write transactions. *i2c_rdwr* is not really a SMBus feature but comes in handy when the master needs to:

1. read or write bulks of data larger than SMBus' 32 bytes limit.
1. write some data and then read from the slave with a repeated start and no stop bit between.

Each operation is represented by a *i2c_msg* message object.


## Example 5: Single i2c_rdwr

```python
from smbus2 import SMBus, i2c_msg

with SMBus(1) as bus:
    # Read 64 bytes from address 80
    msg = i2c_msg.read(80, 64)
    bus.i2c_rdwr(msg)
    
    # Write a single byte to address 80
    msg = i2c_msg.write(80, [65])
    bus.i2c_rdwr(msg)
    
    # Write some bytes to address 80
    msg = i2c_msg.write(80, [65, 66, 67, 68])
    bus.i2c_rdwr(msg)
```

## Example 6: Dual i2c_rdwr

To perform dual operations just add more i2c_msg instances to the bus call:

```python
from smbus2 import SMBus, i2c_msg

# Single transaction writing two bytes then read two at address 80
write = i2c_msg.write(80, [40, 50])
read = i2c_msg.read(80, 2)
with SMBus(1) as bus:
    bus.i2c_rdwr(write, read)
```

## Example 7: Access i2c_msg data

All data is contained in the i2c_msg instances. Here are some data access alternatives.

```python
# 1: Convert message content to list
msg = i2c_msg.write(60, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
data = list(msg)  # data = [1, 2, 3, ...]
print(len(data))  # => 10

# 2: i2c_msg is iterable
for value in msg:
    print(value)

# 3: Through i2c_msg properties
for k in range(msg.len):
    print(msg.buf[k])
```

# Installation instructions

From [PyPi](https://pypi.org/) with `pip`:

```
pip install smbus2
```

From [conda-forge](https://anaconda.org/conda-forge) using `conda`:

```
conda install -c conda-forge smbus2
```

Installation from source code is straight forward:

```
python setup.py install
```
