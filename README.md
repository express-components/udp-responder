# Udp Responder
This library lets you broadcast and listen to messages on a network through UDP.

It can validate a message was sent untampered at a certain time by using SHA256 hashing.

The `UdpResponder` class can take the following options

Property       | Description                                               | Type    | Required | Default
---------------|-----------------------------------------------------------|---------|----------|----------
multicast_addr | The address the UDP command receiver will listen on       | string  | false    | 224.1.1.1
port           | The port the UDP command receiver will listen on          | number  | false    | 6811
secret         | Specify the sha256 signature secret used to sign messages | string  | false    | CHANGEME
ttl            | If using secure the time until a message expires          | integer | false    | 5000

## Properties and Methods
The `UdpResponder` class exposes the following methods and properties.

### UdpResponder::on(event:string, fn:function)
Add an event listener. The following events can occur.

#### opened
```javascript
udpResponder.on('opened', function () {
  console.log('The sender and listener have been opened')
})
```

#### closed
```javascript
udpResponder.on('closed', function () {
  console.log('The sender and listener have been closed')
})
```

#### message
When a message is received it will have a command, possibly some data and information about the sender.

Messages received have their signature validated before being emitted.

The sender is "Remote address information" provided as `rinfo` from the node API.
[https://nodejs.org/api/dgram.html#dgram_event_message](https://nodejs.org/api/dgram.html#dgram_event_message)

```javascript
udpResponder.on('message', function (cmd, data, sender) {
  console.log(`A new ${cmd} message has been received of:`, data)
})
```

#### rejected
See the rejection reasons section below for a list of possible rejection reasons.

Messages rejected have not had a signature validated, and also may not be in any form of readable format.

```javascript
udpResponder.on('rejected', function (reason, message, sender) {
  console.log(`A message was rejected occured:`, reason, cmd)
})
```

### UdpResponder::open()
Start listening for messages on the network

### UdpResponder::close()
Stop listening for messages on the network

### UdpResponder::broadcast(cmd:string, data:any)
Broadcast a message over the network. You must specify a command but data is optional.

## Rejection Reasons
Rejections are returned as an instance of `UdpResponderRejection`.

| Code               | Direction | Description                                                                    |
---------------------|-----------|--------------------------------------------------------------------------------|
| INVALID_FORMAT     | Incoming  | Message received but could not be parsed as it was not in the correct format   |
| INVALID_SIGNATURE  | Incoming  | Message received but had invalid signature.                                    |
| EXPIRED            | Incoming  | Message received but expired ? milliseconds ago.                               |
| INVALID_DATA_TYPE  | Incoming  | Message received with invalid ? content and could not be parsed.               |
| UNKNOWN_DATA_TYPE  | Incoming  | Message received but the data type of ? is unimplemented.                      |

## Example
```javascript
const UdpResponder = require('ec-udp-responder')

const udpResponder = new UdpResponder({
  secret: 'abcdefg'
})

udpResponder.on('opened', async function () {
  udpResponder.broadcast('EXAMPLE-HELLO-COMMAND', {
    name: 'Joe Bloggs',
    hobbies: ['running', 'cycling', 'singing']
  })
})

udpResponder.on('message', async function (msg, sender) {
  if (msg.command === 'EXAMPLE-HELLO-COMMAND') {
    udpResponder.broadcast('EXAMPLE-WELCOME-COMMAND', `A warm welcome to ${msg.name}`)
  }
})

udpResponder.open()

// You can stop everything by calling close
// udpResponder.close()
```
