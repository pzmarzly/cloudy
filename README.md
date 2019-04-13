# C

```bash
doctl compute region list
doctl compute size list
doctl compute ssh-key list
```

## TODO

- `BACKEND=rsync`/`tar` - does anyone need it `tar`?

```bash
# -C / - paths relative to /
# -c   - create
# -z   - compress w/ gzip (.tgz)
# -f - - to stdout/from stdin
# -x   - extract

tar -C / -c --exclude-ignore=.gitignore -z -f - . \
    | ssh $IP tar -C / -x -f -
```

- `SEND_ALL=yes` - send all files

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
