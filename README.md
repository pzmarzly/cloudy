# Cloudy - run your workflow in cloud

Starts up VPS on Digital Ocean or connects to machine you specified, synchronizes current working
directory (`.gitignore`-d files can be excluded), then runs commands (e.g. builds) on VPS.

Also provides shortcut for running X2GO.

Created since it's sometimes not worth it to own expensive hardware (especially laptops).

## Demo

```text
$ cd rust-example
$ cloudy order
Starting cloudy-droplet-eHMSsHFzHFXXMBbm...
Droplet started.
Setting up server...
<cut: apt-get output, rustup output (due to .cloudy-init.sh)>
Rust is installed now. Great!
<cut: more rustup output>
Waiting for files to finish transferring...
Done.
$ cloudy cmd ls -alh
total 20K
drwxr-xr-x 4 root root 4.0K Apr 13 17:45 .
drwx------ 8 root root 4.0K Apr 13 17:44 ..
-rw-r--r-- 1 root root  133 Apr 13 17:42 Cargo.lock
-rw-r--r-- 1 root root  123 Apr 13 18:20 Cargo.toml
drwxr-xr-x 2 root root 4.0K Apr 13 16:42 src
$ cloudy cmd cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 1.86s
$ cloudy cmd ls -alh
total 24K
drwxr-xr-x 4 root root 4.0K Apr 13 17:45 .
drwx------ 8 root root 4.0K Apr 13 17:44 ..
-rw-r--r-- 1 root root  133 Apr 13 17:42 Cargo.lock
-rw-r--r-- 1 root root  123 Apr 13 18:20 Cargo.toml
drwxr-xr-x 2 root root 4.0K Apr 13 16:42 src
drwxr-xr-x 3 root root 4.0K Apr 13 18:20 target
$ cloudy get target/debug/rust-example # you can also grab whole folders
$ target/debug/rust-example
Hello, world!
$ cloudy cmd install_x2go
<cut: apt-get output>
$ cloudy x2go xclock
<cut: pyhoca-cli output>
$ cloudy stop
```

## Installation

Get `cloudy` bash script. Make sure you have bash, rsync, openssl, doctl and Python 3. Then
you are ready to go, but you may want to do more config.

If you want to use `cloudy order`, log in to DigitalOcean CLI using `doctl auth init`.

If you want to use X2GO, install `pyhoca-cli`.

You may also want to set up global `.gitignore` and add `.cloudy-server-running` there. Example:

```bash
git config --global core.excludesfile ~/.gitignore_global
echo \*\*/.cloudy-server-running >> ~/.gitignore_global
```

## Configuration

Cloudy has default configuration that can be overridden by global (`~/.cloudy`) and
local (`.cloudy`) config files. If both files exist, both are loaded, but options from local
config have precedence over global ones.

On server setup and after `cloudy reinit`, `~/.cloudy-init.sh` and `.cloudy-init.sh` are run on
the server as root, if those files exists.

There are currently 2 helper scripts that you can use on remote server - `install_rust` and
`install_x2go`. They are in `PATH`, you can invoke them in `.cloudy-init.sh` or via `cloudy cmd`.

### Default configuration

```bash
# Path to private SSH key, public key
# must be in the same directory and
# end with .pub.
KEY_PATH=~/.ssh/id_rsa
# DigitalOcean region. See list with
# "doctl compute region list"
REGION=fra1
# DigitalOcean VPS specs. See list
# with "doctl compute size list"
SIZE=s-1vcpu-1gb
# Cloudy reuses SSH connections
# for shell commands. It can also
# share this connection with rsync,
# but there may be a rare bug:
# https://github.com/symfony/symfony/issues/25580
# I haven't experienced it, so by default
# it's on.
FILE_CHANNEL_REUSE=1
# Send Cloudy config.
SEND_CLOUDY=0
# Send gitignore-d files.
SEND_GITIGNORE=0
# Send .git directory.
SEND_GIT=0
```

## Commands

```text
$ cloudy # or cloudy help, cloudy -h, cloudy --help
Cloudy - set up VPS and run your workflow on it. Sends your current working
  directory, by default without .gitignore-d files and .git. See project's
  Github page for config.

Subcommands - init:
  order - starts new server on DigitalOcean
  use <[USER@]IP or NAME> - connects to server. NAME must be in ~/.ssh/config
Subcommands - control:
  reinit - reexecutes .cloudy-init.sh on server
  cmd <COMMAND...> - runs command on server. Interactive shell
  cmd-alt <COMMAND...> - runs command on server. Non-interactive shell
  ssh - opens shell on server
  get <PATH> - downloads file off server
  x2go <COMMAND...> - runs GUI command on server, streams back the result
  stop - stops DigitalOcean server
```

## Interactive `cloudy cmd` problem

- `cloudy cmd touch 'a b'` creates 2 files
- `cloudy cmd touch "a b"` creates 2 files
- `cloudy cmd touch a\\ b` creates 1 file `a b`
- `cloudy cmd touch \"a b\"` creates 1 file `a`

Since it is suboptimal, there is `cloudy cmd-alt` subcommand, but it is non-interactive,
meaning STDIN is null, meaning programs cannot ask you for input (so e.g. `apt-get` needs
to be used with `-y` parameter, just like in `.cloudy-init.sh` BTW).

- `cloudy cmd-alt touch 'a b'` creates 2 files
- `cloudy cmd-alt touch "a b"` creates 2 files
- `cloudy cmd-alt touch a\\ b` creates 1 file `a b`
- `cloudy cmd-alt touch \"a b\"` creates 1 file `a b`

I recommend using `cloudy cmd` and `cloudy cmd-alt` only for small commands, to invoke some
processes (e.g. start build scripts). Use `cloudy ssh` if you need more.

## When things go wrong

Make sure you aren't left with zombie DigitalOcean servers:

```bash
doctl compute droplet list
doctl compute droplet delete <ID>
```

## TODO

- automatically send new files (right now `rsync` is run on every `cloudy` invocation, but nobody
  would like to reset X11 programs when they want to push new files. Workaround: `cloudy cmd true`
  to push files)
- add option to use `mosh`
- search parent directories for `.cloudy-server-running`
- global `.cloudy-server-running` database as an option
- use `curl` instead of `doctl`
- `tar` instead of `rclone` - probably unnecessary

```bash
# -C / - paths relative to /
# -c   - create
# -z   - compress w/ gzip (.tgz)
# -f - - to stdout/from stdin
# -x   - extract

tar -C / -c --exclude-ignore=.gitignore -z -f - . \
    | ssh $IP tar -C / -x -f -
```

## My other projects

- [mic_over_Mumble](https://github.com/pzmarzly/mic_over_mumble) - use your phone as Linux microphone
- [x11-input-supercharger](https://github.com/pzmarzly/x11-input-supercharger) - middle-mouse-click
  scrolling mode, conditional key rebinding when using Wacom tablet
- [x11-input-mirror](https://github.com/pzmarzly/x11-input-mirror) - broadcast X11 input events,
  replay them real-time on other machines
- [movie library](https://github.com/movie-rs/movie) - actor system for Rust
- [portforwarder-rs](https://github.com/pzmarzly/portforwarder-rs) - CLI port opener for
  UPnP-enabled routers
- [Raspberry Pi video grabbing and livestreaming](https://pzmarzly.pl/2018/03/17/raspberry-pi-livestreaming.html)
- [yoke - Android as Linux gamepad](https://github.com/rmst/yoke)
- [sshfs manager](https://github.com/pzmarzly/sshfs-manager) - alpha

## Resources used

- [bash subcommands](https://gist.github.com/waylan/4080362)
- [SSH key digest](https://serverfault.com/a/775193/449626)
- [bash is_set](https://stackoverflow.com/a/46704718/5108318)
- [run script remotely](https://stackoverflow.com/a/2732991/5108318)
- [rsync exclude](https://stackoverflow.com/a/15373763/5108318)
- [Python JSON parse](https://stackoverflow.com/a/8400375/5108318)
- [bash arrays](https://stackoverflow.com/a/19122890/5108318)
- ["ttyname failed" Ubuntu bug](https://superuser.com/a/1160074/620906)
- [SSH multiplexing](https://stackoverflow.com/a/20410383/5108318)
- [bash arrays - push_back](https://stackoverflow.com/a/1951523/5108318)
- [bash OR](https://stackoverflow.com/a/8972266/5108318)
- [bash default argument value](https://stackoverflow.com/a/9333006/5108318)
