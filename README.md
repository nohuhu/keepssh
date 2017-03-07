# keepssh

Shell script that keeps SSH connections open

This is somewhat like autossh but written as portable Bourne shell script,
which makes life a bit easier to us poor folks who have to deal with old
UNIX systems.

# What it does

Keeps a SSH connection open no matter what, reconnecting when necessary.
When port forwarding is requested, `keepssh` will make sure it is successful
as well. If it's not we will drop the session and reconnect until forwarding
is up, with special provisions for older OpenSSH clients that can't monitor
port forwarding themselves.

This is of course mostly useful for setting up tunnels but `keepssh` can be
used interactively too; works pretty well on a laptop when you need to move
around. Close the lid, connection times out; open the lid, it is back up.

# Usage

`keepssh` is intended to be easy to use. Run it as you would run `ssh`,
there are no special arguments; e.g.:

    keepssh -o foo=yes -i ~/.ssh/identity -p 2222 foo@bar.baz

Instead of:

        ssh -o foo=yes -i ~/.ssh/identity -p 2222 foo@bar.baz

All arguments are passed through to `ssh` transparently, except `-f`
and `-q`; these are handled internally to produce the same behavior:

- `-f` will tell `keepssh` to drop into background; combined with
`KEEPSSH_PIDFILE` variable (see below) this allows for easier
scripting because ssh can be restarted many times and its PID will
change every time but `keepssh` process will stay the same until
terminated.

- `-q` will tell `keepssh` to be quiet. No errors or warnings are
printed to stderr, just like `ssh` does.

# Environment variables

There are a few extra features and behaviors controlled via env
variables:

- `KEEPSSH_SSH=<path>` will force using SSH client at the specified path
instead of the default `ssh`.

- `KEEPSSH_VERBOSE=<level>` will make `keepssh` log its actions on stderr.
Values of <level> can be 1 or 2, with greater verbosity at 2.

- `KEEPSSH_DEBUG=<level>` will add debugging information to usual log
messages. Level=1 will `set -x` and level=2 will `set -xv`, which is a lot
of output. When debugging is requested, `-q` option is ignored.

- `KEEPSSH_TIMEOUT=<seconds>` will change the timeout between reconnect
attempts (default is 5). If the time since last reconnect attempt is less
than this value, the timeout is multiplied by 2 for each subsequent failed
attempt, until `KEEPSSH_MAX_TIMEOUT` is reached. The timeout will be reset
back to original `KEEPSSH_TIMEOUT` value after a successful connection.
This is useful for preventing network flooding.

- `KEEPSSH_MAX_TIMEOUT=<seconds>` will change the maximum timeout betweeen
reconnect attempts (default is 300).

- `KEEPSSH_PIDFILE=<path>` will print the PID of the process to specified
file. Use this process id to stop `keepssh`, like this:

    kill `cat /tmp/keepssh.pid`

- `KEEPSSH_LOGFILE=<path>` will divert stderr to the specified file,
including ssh output on stderr. When both log file and `-q` option
are used, `keepssh` will create the file but won't print anything
in it.

- `KEEPSSH_FIFO=<path>` will use the path (or mktemp pattern)
to create named pipe where ssh stderr will be redirected. This is
used on old systems where OpenSSH doesn't support ExitOnForwardFailure
option and we have to parse its stderr for errors.
Default is whatever your local `mktemp` comes up with.

# Installation

There are no special instructions for `keepssh`, it is designed to be
simple and portable. Drop the script into your ~/bin and run
`chmod +x ~/bin/keepssh` and you're all set. Something like this:

    curl -L https://raw.github.com/nohuhu/keepssh/master/keepssh > ~/bin/keepssh
    chmod 755 ~/bin/keepssh

# Requirements

Obviously you will need a SSH client to use this script. So far
`keepssh` was tested with OpenSSH in Linux, Mac OS X and Solaris 10;
there is a reasonable expectation that it might work in other
systems, too.

I would very much like to make `keepssh` compatible with any UNIX
system out there but I don't have access to anything beyond listed
above. Patches and bug reports are always welcome.

Please report issues here: https://github.com/nohuhu/keepssh/issues

