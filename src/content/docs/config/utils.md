---
title: Utils
description: Utility functions
---

## Running external commands

### Synchronously

```ts
function exec(cmd: string): string
```

This is synchronous, meaning it will **block the eventloop**

```js
const echo = Utils.exec('echo "Hi Mom"') // returns string
console.log(echo) // logs "Hi Mom"
```

```js
const uptime = Utils.exec(`bash -c "uptime | awk '{print $3}' | tr ',' ' '"`)
console.log(uptime)
```

### Asynchronously

```ts
function execAsync(cmd: string | string[]): Promise<string>;
```

This won't block,

```js
Utils.execAsync(['echo', 'Hi Mom'])
    .then(out => print(out))
    .catch(err => print(err));
```

:::note
Both `exec` and `execAsync` launches the given binary on its own,
meaning if you want to use `|` pipes or any other shell features
then you have to run it with bash.

```js
Utils.execAsync(['bash', '-c', 'cmd | cmd && cmd']);
Utils.exec('bash -c "cmd | cmd && cmd"');
```

:::

## Running external scripts

```ts
function subprocess(
    cmd: string | string[],
    callback: (out: string) => void,
    onError = logError,
    bind?: Gtk.Widget,
): Gio.Subprocess
```

Takes two to four arguments, returns [Gio.Subprocess](https://gjs-docs.gnome.org/gio20~2.0/gio.subprocess)

```js
const proc = Utils.subprocess(
    // command to run, in an array just like execAsync
    ['bash', '-c', 'path-to-bash-script'],

    // callback when the program outputs something to stdout
    (output) => print(output),

    // callback on error
    (err) => logError(err),

    // optional widget parameter
    // if the widget is destroyed the subprocess is forced to quit
    widget,
)
```

Killing the process

```js
proc.force_exit()
```

## Writing and reading files

```ts
function readFile(file: string | Gio.File): string
function readFileAsync(file: string | Gio.File): Promise<string>

function writeFileSync(string: string, path: string): Gio.File
function writeFile(string: string, path: string): Promise<Gio.File>
```

### Synchronously

```js
const contents = Utils.readFile('/path/to/file')

Utils.writeFileSync('/path/to/file', 'some content')
```

### Asynchronously

```js
Utils.readFileAsync('path-to-file')
    .then(content => print('contents of the file: ' + content))
    .catch(logError)

Utils.writeFile('Contents: Hi Mom', 'path-to-file')
    .then(file => print('file is the Gio.File'))
    .catch(err => print(err))
```

## Monitoring files and directories

```ts
function monitorFile(
    path: string,
    callback?: (file: Gio.File, event: Gio.FileMonitorEvent) => void,
    flags = Gio.FileMonitorFlags.NONE,
): Gio.FileMonitor | null
```

```js
const monitor = Utils.monitorFile('/path/to/file', (file, event) => {
    print(Utils.readFile(file), event)
})
```

### Canceling the monitor

```js
monitor.cancel()
```

## Timeout and Interval

You can use native JS `setTimeout` and `setInterval`, they return a [GLib.Source](https://docs.gtk.org/glib/struct.Source.html)

Timeout

```js
const source = setTimeout(() => { /* callback */ }, 1000)
```

Interval

```js
const source = setInterval(() => { /* callback */ }, 1000)
```

To cancel them use `GLib.Source.destroy`

```js
source.destroy()
```

You can use the ones from `Utils`

```ts
function interval(
    interval: number,
    callback: () => void,
    bind?: Gtk.Widget,
): number

function timeout(
    ms: number,
    callback: () => void,
): number
```

```js
const id = Utils.timeout(1000, () => {
  // runs with a second delay
})
```

The widget parameter is optional

```js
const id = Utils.interval(1000, () => {
  // runs immediately and once every second
})
```

If you pass a widget to `Utils.interval`, it will automatically be canceled,
when the widget is destroyed

```js
const widget = Widget.Label()
const id = Utils.interval(1000, () => {}, label)
widget.destroy()
```

To cancel them use `GLib.source_remove`

```js
import GLib from 'gi://GLib'
GLib.source_remove(id)
```

## Lookup an Icon name

```js
const icon = Utils.lookUpIcon('dialog-information-symbolic')

if (icon) {
    // icon is the corresponding Gtk.IconInfo
}
else {
    // null if it wasn't found in the current Icon Theme
}
```

## Fetch

should be pretty close to the web api

```js
Utils.fetch('http://wttr.in/?format=3')
    .then(res => res.text())
    .then(print)
    .catch(console.error)
```

## Authentication

authenticate a user using pam

:::note
on NixOS make sure you have `security.pam.services.ags = {}` in `configuration.nix`
:::

```js
Utils.authenticate('password')
    .then(() => print('authentication sucessful'))
    .catch(err => logError(err, 'unsucessful'))

Utils.authenticateUser("username", "password")
    .then(() => print("authentication sucessful"))
    .catch(err => logError(err, 'unsucessful'))
```

## Send Notifications

```js
Utils.notify('summary', 'body', 'icon-name')
```

```js
const id = await Utils.notify({
    summary: 'Title',
    body: 'Description',
    iconName: 'icon-name',
    actions: {
        'Click Me': () => print('clicked')
    }
})

const notifications = await Service.import('notifications')
const n = notifications.getNotification(id)
```
