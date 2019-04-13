# Cloudy - run your workflow in cloud

Starts up VPS on Digital Ocean or connects to machine you specified, synchronizes current working
directory (`.gitignore`-d files can be excluded), then runs commands (e.g. builds) on VPS.
Created since it's sometimes not worth it to buy expensive hardware (especially laptops).

## Demo

```text
$ cd rust-example
$ cloudy order
Starting cloudy-droplet-eHMSsHFzHFXXMBbm...
Droplet started.
Setting up server...
<cut>
Rust is installed now. Great!
<cut>
Waiting for files to finish transferring...
Done.
$ cloudy cmd ls -alh
total 24K
drwxr-xr-x 4 root root 4.0K Apr 13 17:45 .
drwx------ 8 root root 4.0K Apr 13 17:44 ..
-rw-r--r-- 1 root root  133 Apr 13 17:42 Cargo.lock
-rw-r--r-- 1 root root  123 Apr 13 18:20 Cargo.toml
drwxr-xr-x 2 root root 4.0K Apr 13 16:42 src
$ cloudy cmd cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
$ cloudy cmd ls -alh
total 24K
drwxr-xr-x 4 root root 4.0K Apr 13 17:45 .
drwx------ 8 root root 4.0K Apr 13 17:44 ..
-rw-r--r-- 1 root root  133 Apr 13 17:42 Cargo.lock
-rw-r--r-- 1 root root  123 Apr 13 18:20 Cargo.toml
drwxr-xr-x 2 root root 4.0K Apr 13 16:42 src
drwxr-xr-x 3 root root 4.0K Apr 13 18:20 target
$ cloudy get target/debug/rust-example
$ target/debug/rust-example
Hello, world!
$ cloudy stop
```

## Installation

Get `cloudy` bash script. Make sure you have bash, scp, rsync, openssl, doctl and Python 3.

If you want to use `cloudy order`, log in to DigitalOcean CLI using `doctl auth init`.

You are ready to go, but you may want to do the configuration.

## Configuration

Cloudy has default configuration that can be overridden by global (`~/.cloudy`) and
local (`.cloudy`) config files. If both files exist, both are loaded, but options from local
config have precedence over global ones.

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
# share this connection with rsync
# and scp, but there may be a rare bug:
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

## TODO

- add option to use `mosh`
- `cloudy get` directories
- `tar` instead of scp or rclone

```bash
# -C / - paths relative to /
# -c   - create
# -z   - compress w/ gzip (.tgz)
# -f - - to stdout/from stdin
# -x   - extract

tar -C / -c --exclude-ignore=.gitignore -z -f - . \
    | ssh $IP tar -C / -x -f -
```

## Resources used

- https://gist.github.com/waylan/4080362
- https://serverfault.com/a/775193/449626
- https://stackoverflow.com/a/46704718/5108318
- https://stackoverflow.com/a/2732991/5108318
- https://stackoverflow.com/a/15373763/5108318
- https://stackoverflow.com/a/8400375/5108318
- https://stackoverflow.com/a/19122890/5108318
- https://superuser.com/a/1160074/620906
- https://stackoverflow.com/a/20410383/5108318
- https://stackoverflow.com/a/1951523/5108318
