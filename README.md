# modbusd 

[![Build Status](https://travis-ci.org/taka-wang/modbusd.svg?branch=dev)](https://travis-ci.org/taka-wang/modbusd) 
[![GitHub tag](https://img.shields.io/github/tag/taka-wang/modbusd.svg)](https://github.com/taka-wang/modbusd/tags) 
[![Release](https://img.shields.io/github/release/taka-wang/modbusd.svg)](https://github.com/taka-wang/modbusd/releases/latest)
[![Join the chat at https://gitter.im/taka-wang/modbusd](https://badges.gitter.im/taka-wang/modbusd.svg)](https://gitter.im/taka-wang/modbusd?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Modbus master daemon 

- Support doxygen style comments.
- ZMQ is a high-level message library, you can plug in your own socket implemetations without losing the core functionalities.

# TOC

- [Design](#design)
- [Setup](#setup)
- [Continuous Integration](#ci)
- [Documentation](#doc)

---

<a name="design"></a>
# Design

## Implemented libmodbus function codes

| FC    | Description            | Max Len | API                                                                                                                                                 |
|:-----:|------------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------------|
| 0x01  | read coils             |  2000   |[int modbus_read_bits(modbus_t *ctx, int addr, int nb, uint8_t *dest)](http://libmodbus.org/docs/v3.1.4/modbus_read_bits.html)                       |  
| 0x02  | read discrete input    |  2000   |[int modbus_read_input_bits(modbus_t *ctx, int addr, int nb, uint8_t *dest)](http://libmodbus.org/docs/v3.1.4/modbus_read_input_bits.html)           |
| 0x03  | read holding registers |  125    |[int modbus_read_registers(modbus_t *ctx, int addr, int nb, uint16_t *dest)](http://libmodbus.org/docs/v3.1.4/modbus_read_registers.html)            |
| 0x04  | read input registers   |  125    |[int modbus_read_input_registers(modbus_t *ctx, int addr, int nb, uint16_t *dest)](http://libmodbus.org/docs/v3.1.4/modbus_read_input_registers.html)|
| 0x05  | write single coil      |   -     |[int modbus_write_bit(modbus_t *ctx, int addr, int status)](http://libmodbus.org/docs/v3.1.4/modbus_write_bit.html)                                  |
| 0x06  | write single register  |   -     |[int modbus_write_register(modbus_t *ctx, int addr, int value)](http://libmodbus.org/docs/v3.1.4/modbus_write_register.html)                         |
| 0x0F  | write multi coils      |  1968   |[int modbus_write_bits(modbus_t *ctx, int addr, int nb, const uint8_t *src)](http://libmodbus.org/docs/v3.1.4/modbus_write_bits.html)                |
| 0x10  | write multi registers  |  125    |[int modbus_write_registers(modbus_t *ctx, int addr, int nb, const uint16_t *src)](http://libmodbus.org/docs/v3.1.4/modbus_write_registers.html)     |

## coil/register number and address table

|Coil/Register numbers|data address       |type          |table name                     |offset| function code|
|:--------------------|:------------------|:-------------|:------------------------------|:-----|:-------------|
|1-9999               |0000 to 270E (9998)|Read-Write    |Discrete Output Coils          |1     | 1, 5, 15     |
|10001-19999          |0000 to 270E (9998)|Read-Only     |Discrete Input Contacts        |10001 | 2            |
|30001-39999          |0000 to 270E (9998)|Read-Only     |Analog Input Registers         |30001 | 4            |
|40001-49999          |0000 to 270E (9998)|Read-Write    |Analog Output Holding Registers|40001 | 3, 6, 16     |
---

## Configuration file format
```javascript
{
    "syslog": 1,
    "ipc_sub": "ipc:///tmp/to.modbus",
    "ipc_pub": "ipc:///tmp/from.modbus",
    "mbtcp_connect_timeout": 200000
}
```

## Modbus TCP Json command format

### -> mbtcp read request
```javascript
{
	"ip": "192.168.3.2",
	"port": "502",
	"slave": 22,
	"tid": 1,
	"cmd": "fc1",
	"addr": 250,
	"len": 10
}
```

### <- mbtcp read reponse
```javascript
{
	"tid": 1,
	"data": [1,2,3,4],
	"status": "ok"
}
```

### -> mbtcp write request
```javascript
{
	"ip": "192.168.3.2",
	"port": "502",
	"slave": 22,
	"tid": 1,
	"cmd": "fc5",
	"addr": 80,
	"len": 4,
	"data": [1,2,3,4]
}
```

### <- mbtcp write response
```javascript
{
	"tid": 1,
	"status": "ok"
}
```

### -> mbtcp set timeout
```javascript
{
	"tid": 1,
	"cmd": "timeout",
	"data": 210000
}
```

### <- mbtcp set timeout response
```javascript
{
	"tid": 1,
	"status": "ok"
}
```

## External libraries

- [libmodbus](http://libmodbus.org)
- [libzmq](https://github.com/zeromq/libzmq)
- [czmq](https://github.com/zeromq/czmq)
- [uthash](https://troydhanson.github.io/uthash)
- [cJSON](https://github.com/DaveGamble/cJSON)

---

## Library documentations

- [uthash user guide](http://troydhanson.github.io/uthash/userguide.html)
- [libmodbus api document](http://libmodbus.org/docs/v3.1.4/)
- [libmodbus header](https://github.com/stephane/libmodbus/blob/master/src/modbus.h)
- [cJSON examples](https://github.com/DaveGamble/cJSON)


## Flow Chart

![flow](image/flow.png)

---

<a name="setup"></a>
# Setup

Step by step from scratch or ([Travis CI](https://travis-ci.org) + [Docker](#ci))

## Setup development dependencies

```bash
sudo apt-get update
sudo apt-get install -y git build-essential autoconf libtool pkg-config cmake
```

---

## Setup OSS libs dependencies

### Install libmodbus library (3.1.4)

```bash
git clone https://github.com/stephane/libmodbus/
cd libmodbus
./autogen.sh
./configure
make
sudo make install
sudo ldconfig
```

### Install libzmq (3.2.5)

```bash
wget https://github.com/zeromq/zeromq3-x/releases/download/v3.2.5/zeromq-3.2.5.tar.gz
tar xvzf zeromq-3.2.5.tar.gz
cd zeromq-3.2.5
./configure
make
sudo make install
sudo ldconfig
```

### Install czmq (high-level C binding for zeromq)

```bash
git clone git://github.com/zeromq/czmq.git
cd czmq
./autogen.sh
./configure
make
sudo make install
sudo ldconfig
```

## Setup testing environment

### Install node.js 4.x & zmq binding on ubuntu

```bash
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g zmq
```
---

## Build
```bash
git clone modbusd
cd modbusd
mkdir build
cd build
cmake ..
make
./modbusd ../modbusd.json # load external configuration file
```
---

<a name="ci"></a>
# Continuous Integration

We do continuous integration and update docker images after git push by [Travis CI](https://travis-ci.org/taka-wang/modbusd).

![ci](image/ci.png)

## // x86_64 platform

### Docker base images
- [x86_64 git repo](https://github.com/taka-wang/docker-ubuntu)
- [docker hub](https://hub.docker.com/u/takawang/)

### Docker images registry

You can download pre-built docker images according to the following commands.

- docker pull [takawang/modbus-server](https://hub.docker.com/r/takawang/modbus-server/)
- docker pull [takawang/modbus-zclient](https://hub.docker.com/r/takawang/modbus-zclient/)
- docker pull [takawang/modbusd](https://hub.docker.com/r/takawang/modbusd/)


### Docker images and testing from the scratch
```bash
# build simulation server image
docker build -t takawang/modbus-server tests/mbserver/.
# build zclient image
docker build -t takawang/modbus-zclient tests/zclient/.
# build modbusd image
docker build -t takawang/modbusd .

# run modbus server
docker run -itd --name=slave takawang/modbus-server
# run modbusd
docker run -v /tmp:/tmp --link slave -it --name=modbusd takawang/modbusd
# run zclient
docker run -v /tmp:/tmp -it --link slave takawang/modbus-zclient
```

### Docker composer
```bash
# build & run
docker-compose up 
# exit test
ctrl+c
```

## // armhf

### Docker base images
- [armhf git repo](https://github.com/taka-wang/docker-armv7)
- [docker hub](https://hub.docker.com/u/takawang/)

### Docker images registry

You can download pre-built docker images according to the following commands.

- docker pull [takawang/arm-modbus-server](https://hub.docker.com/r/takawang/arm-modbus-server/)
- docker pull [takawang/arm-modbus-zclient](https://hub.docker.com/r/takawang/arm-modbus-zclient/)
- docker pull [takawang/arm-modbusd](https://hub.docker.com/r/takawang/arm-modbusd/)


### Docker images and testing from the scratch
```bash
# build simulation server image
docker build -t takawang/arm-modbus-server -f tests/mbserver/Dockerfile.arm .
# build zclient image
docker build -t takawang/arm-modbus-zclient tests/zclient/Dockerfile.arm .
# build modbusd image
docker build -t takawang/arm-modbusd -f Dockerfile.arm .

# run modbus server
docker run -itd --name=slave takawang/arm-modbus-server
# run modbusd
docker run -v /tmp:/tmp --link slave -it --name=modbusd takawang/arm-modbusd
# run zclient
docker run -v /tmp:/tmp -it --link slave takawang/arm-modbus-zclient
```

## Deployment Diagram

![deployment](image/deployment.png)

---

<a name="doc"></a>
# Documentations

- [API Document](http://taka-wang.github.io/modbusd)