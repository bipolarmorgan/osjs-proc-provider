<p align="center">
  <img alt="OS.js Logo" src="https://raw.githubusercontent.com/os-js/gfx/master/logo-big.png" />
</p>

[OS.js](https://www.os-js.org/) is an [open-source](https://raw.githubusercontent.com/os-js/OS.js/master/LICENSE) desktop implementation for your browser with a fully-fledged window manager, Application APIs, GUI toolkits and filesystem abstraction.

[![Build Status](https://travis-ci.org/os-js/osjs-proc-provider.svg?branch=master)](https://travis-ci.org/os-js/osjs-proc-provider)
[![Support](https://img.shields.io/badge/patreon-support-orange.svg)](https://www.patreon.com/user?u=2978551&ty=h&u=2978551)
[![Back](https://opencollective.com/osjs/tiers/backer/badge.svg?label=backer&color=brightgreen)](https://opencollective.com/osjs)
[![Sponsor](https://opencollective.com/osjs/tiers/sponsor/badge.svg?label=sponsor&color=brightgreen)](https://opencollective.com/osjs)
[![Donate](https://img.shields.io/badge/liberapay-donate-yellowgreen.svg)](https://liberapay.com/os-js/)
[![Donate](https://img.shields.io/badge/paypal-donate-yellow.svg)](https://paypal.me/andersevenrud)
[![Community](https://img.shields.io/badge/join-community-green.svg)](https://community.os-js.org/)

# OS.js v3 Proc Service Provider

Adds support for spawning and piping processes and pseudo terminals on the node server.

Communicates via the internal websocket.

## Installation

```bash
npm install --save --production @osjs/proc-provider
```

In your initialization scripts:

```javascript
// Client index.js file
import {ProcServiceProvider} from '@osjs/proc-provider';
core.register(ProcServiceProvider);

// Client index.scss file
@import "~@osjs/proc-provider/dist/main.css";

// Server index.js file
const {ProcServiceProvider} = require('@osjs/proc-provider/src/server.js');
core.register(ProcServiceProvider);
```

## Configuration

By default the server provider is set up to only allow users with the `admin` group to access this feature.

You can change this by adding options:

```javascript
const {ProcServiceProvider} = require('@osjs/proc-provider/src/server.js');
core.register(ProcServiceProvider, {
  args: {
    groups: ['other-group']
  }
});
```

## API

You can reach this service with `core.make('osjs/proc')`.

* `pty(cmd, ...args) => Promise<p, Error>` - Creates a new pseudo terminal over websocket
* `spawn(cmd, ...args) => Promise<p, Error>` - Spawns a new process over websocket
* `exec(cmd, ...args) => Promise<{code, stdout, stderr}, Error>` - Execs a process over http

The `cmd` can either be a string, or an object: `{cmd: string, env: {A: 'value'}}`.

The `p` returned in a promise resolution is an `EventEmitter` with some special methods for interacting with the process.

### Pseudo terminal over websocket

Execute a process, but inside a PTY. This makes it possible to use interactive processes.

*Note that it is not possible to differentiate between stdout and stderr in this case.*

```javascript
core.make('osjs/proc')
  .pty('ls')
  .then(p => {
    // Process events
    p.on('data', str => console.log(str))
    p.on('exit', code => console.info(code))

    // Internal events
    p.on('spawned', () => {}); // Spawn successful
    p.on('error', error => console.error(error)); // Spawn errors

    // Send data to the shell
    p.send('Hello World\n');

    // You can kill long running processes
    p.kill();
  })
```

### Execute over websocket

Execute a process via standard node child_process.

*Note that processes that requires an interactive shell won't work here. See PTY above.*

```javascript
core.make('osjs/proc')
  .spawn('ls')
  .then(p => {
    // Process events
    p.on('stdout', str => console.log(str))
    p.on('stderr', str => console.warn(str))
    p.on('exit', code => console.info(code))

    // Internal events
    p.on('spawned', () => {}); // Spawn successful
    p.on('error', error => console.error(error)); // Spawn errors

    // You can kill long running processes
    p.kill();
  })
```

### Execute over http

Directly execute a program via child_process and return the result.

```javascript
core.make('osjs/proc')
  .exec('ls')
  .then(({stdout, stderr, code}) => console.log(stdout, stderr, code))
```

### Passing arguments

```javascript
core.make('osjs/proc')
  .spawn('ls', '-l') // Works for all methods
```

### Passing environmental variables

You can also pass environmental data to all methods.

```javascript
core.make('osjs/proc')
  .spawn({cmd: 'ls', env: {foo: 'bar'}}) // Works for all methods
```

### Spawning windows to monitor output

For a PTY this also supports input. Just spawn a process as above, but:

```javascript
core.make('osjs/proc')
  .pty('ls')
  .then(p => {
    const win = p.createWindow({
      // Keep window open for as long as you want
      keepOpen: false
    })

    // Close window after 2.5s when command is complete
    p.on('exit', () => setTimeout(() => win.destroy(), 2500))
  })
```

## Features

- Launch processes via websocket (pipeable)
- Launch pseudo terminals via websocket (pipeable)
- Launch process via http
- Support for manually killing a running process
- Server clears out stale processes (ex when client disconnects)
- Works on all platforms

## TODO

- [ ] Add option for spawning standalone socket
- [ ] Move the HTTP API purely to websocket signals ?
- [ ] Add a system socket and host service
- [ ] Add `shell` method for direct PTY

## Contribution

* **Become a [Patreon](https://www.patreon.com/user?u=2978551&ty=h&u=2978551)**
* **Support on [Open Collective](https://opencollective.com/osjs)**
* [Contribution Guide](https://github.com/os-js/OS.js/blob/v3/CONTRIBUTING.md)

## Documentation

See the [Official Manuals](https://manual.os-js.org/v3/) for articles, tutorials and guides.

## Links

* [Official Chat](https://gitter.im/os-js/OS.js)
* [Community Forums and Announcements](https://community.os-js.org/)
* [Homepage](https://os-js.org/)
* [Twitter](https://twitter.com/osjsorg) ([author](https://twitter.com/andersevenrud))
* [Google+](https://plus.google.com/b/113399210633478618934/113399210633478618934)
* [Facebook](https://www.facebook.com/os.js.org)
* [Docker Hub](https://hub.docker.com/u/osjs/)
