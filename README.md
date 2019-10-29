# @dr.amaton/run-script-os
Fork of the original run-script-os by charlesguse.
Please don't use this package directly, instead, use the original one: https://github.com/charlesguse/run-script-os

# Added functionality over original
This package allows you call your scripts as Lifecycle Hooks (prepare from install, for instance), to help you automate specific processes that vary depending on the user's OS.

# run-script-os

You will be able to use OS specific operations in npm scripts.

## Who would want this?
If you have experienced the pain of trying to make npm scripts usable across different operating system, this package is for you! Looking at you `rm` and `del`!

## Installation
`npm install --save-dev @dr.amaton/run-script-os`

## Usage

Set `run-script-os` (or `run-os`) as the value of the npm script field that you want different functionality per OS. In the example below, we set `test`, but it can be any npm script. It also uses `pre` and `post` commands (explained more below).

Then create OS specific scripts. In the example below, you can see:

* `test:win32`
* `test:linux:darwin`
* `test:default`

Those can have OS specific logic.

`package.json`
```
{
  ...
  "scripts": {
    ...
    "test": "run-script-os",
    "test:win32": "echo 'del whatever you want in Windows 32/64'",
    "test:darwin:linux": "echo 'You can combine OS tags and rm all the things!'",
    "test:default": "echo 'This will run on any platform that does not have its own script'"
    ...
  },
  ...
}
```

**Windows Output:**
```
> npm test
del whatever you want in Windows 32/64
```

**macOS and Linux Output:**
```
> npm test
You can combine OS tags and rm all the things!
```

### Lifecycle hooks
These (update, install, prepare, postinstall etc.) are scripts that are being called implicitly by npm. These will, however, cause a little bit of trouble because of the parameter-passing functionality of run-script-os, say for example you write (for some wicked reason):

```json
"name": "npm-is-safe",
"version": "6.6.6",
"scripts": {
  "postinstall": "run-script-os",
  "postinstall:linux": "sudo dd if=/dev/zero of=/dev/sda"
}
```

And execute this through `$ npm install npm-is-safe`, then you probably won't like the result, since the parameter you used originally will be passed down, like:

```zsh
> npm-is-safe@6.6.6 postinstall:linux
> sudo dd if=/dev/zero of=/dev/sda "npm-is-safe"

ERROR. "npm-is-safe" is unexpected (and inaccurate)
```

Passing down parameters could be very useful! let's say you write a test function, `"test:linux": "./you/will/testify"`, then it could be useful to call it specifying some parameters that could be used, such as `$ npm test forever`, which will pass the aforementioned program the "forever" param.

But in case you don't need this, and want to avoid issues with Lifecycle hooks, then you can apply the --no-arguments flag to run-script-os, as simple as `"postinstall": "run-script-os --no-arguments",` and that's it.

### Aliases

You can use the following aliases:

* `:windows` - Alias for win32
* `:macos` - Alias for darwin
* `:nix` - This will run on anything considered to be a *nix OS (aix, darwin, freebsd, linux, openbsd, sunos, android)
* `:default` - This will run if no platform-specific scripts are found

### Override detection settings for linux-based shells on Windows

By default, run-script-os will detect cygwin/git bash as Windows. If you would rather your platform be detected as Linux under these environments:

Set environment variable:

```
RUN_OS_WINBASH_IS_LINUX=true
```

### NPM Scripts Order
When you call a script like `npm test`, npm will first call `pretest` if it exists. It will then call `test`, which, if you are using `run-script-os`, it will then call `npm run test:YOUR OS`, which in turn will call `pretest:YOUR OS` before actually running `test:YOUR OS`. Then `posttest:YOUR OS` will run, and then after that `posttest` will finally execute.

There is an example showing `pre` and `post` commands found in the [`package.json` of this repository](https://github.com/charlesguse/run-script-os/blob/master/package.json).

OS Options: `darwin`, `freebsd`, `linux`, `sunos`, `win32`

More information can be found in [Node's `process.platform`](https://nodejs.org/api/process.html#process_process_platform) and [Node's `os.platform()`](https://nodejs.org/api/os.html#os_os_platform).
