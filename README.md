# rmt - run commands on the remote end of a network-mounted filesystem

`rmt` is a bash script which figures out whether the current directory is part of a network-mounted
filesystem (`nfs` and `sshfs` are currently supported) and runs a command or shell on that remote
machine in the corresponding directory.

You might immediately wonder _why_ use `rmt` instead of e.g. `ssh`ing into the remote machine. This is
entirely a matter of preference, but for certain tasks I like files to be mounted over the network
so I can manipulate them "locally" (and use local editors, for instance). Occasionally I want to run
commands on the remote end, and this script makes it very easy to do so without breaking context
(retaining the correct directory). This works best over a local trusted network (for instance, if
using a local separate build machine/server), since the experience can be seamless with low latency
and password-less ssh keys to authenticate.

`rmt` uses `findmnt` to detect whether the current directory is part of a network mount, where the
mount is rooted, and what the remote host and directory is. It uses this information to form an
`ssh` command line. It also has some optional optimizations to cache a remote interactive shell
environment to reduce latency, which is useful if you're going to be frequently running commands.

## Requirements

* `bash`
* `findmnt` (part of the `util-linux` package)
* `ssh`

Because of the use of `findmnt` only Linux is currently supported (though I believe macos support
may be possible).

## Installation

After ensuring you have the above requirements, simply add the `bin` directory to your `PATH` (or,
if you prefer, copy `rmt` to a directory on your `PATH`). You must also set up ssh login to the
remote machines which are mounted.

## Usage

There are a few ways to use `rmt`. In all cases, commands/shells are run in the remote directory
corresponding to the current directory.

Similar to `ssh`,
* if invoked without arguments, `rmt` launches a login shell; and
* if invoked with arguments, the arguments form the command to run remotely.

You can also symlink to `rmt`. If you do this (and place the files in a directory on your `PATH`),
`rmt` will use the name of the symlink as the remote command to run, and remaining arguments as the
arguments to the command. This works especially well with tools like [`direnv`][], since you can add
a directory with many symlinked commands to the `PATH` when in the mounted filesystem.

If you define a `RMT_CACHE_ENV` environment variable, `rmt` will cache the environment of a single
interactive shell and reuse it (using the value of the variable as a cache key). This reduces
latency when frequently running short commands. It also is _generally_ safe to do as long as the
remote end has a consistent interactive shell environment (i.e. one that isn't going to change based
on time or other environmental factors). If you want to delete (and thus refresh) the cache, run
`rmt --recache` (with `RMT_CACHE_ENV` set). `RMT_CACHE_ENV` may contain path separators (though this
will create directories).

[`direnv`]: https://direnv.net/
