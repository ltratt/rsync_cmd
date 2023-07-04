# rsync_cmd

`rsync_cmd` is designed to make it easy to develop a project locally, but run
it remotely. It uses [rsync](https://rsync.samba.org/) to mirror the current
working directory to a remote and `ssh` to run a command remotely in that
directory. Its usage is as follows:

```
rsync_cmd [-d|--dry-run] [--ignore-conf-file] [-v] \
  [-- <rsync_opt_1> .. <rsync_opt_n> --] \
  <remote host> [<arg_1> ... <arg_n>]
```

For example:

```
$ cd ~/src/yk/
$ rsync_cmd ssh.example.com cargo test
```

This will take the contents of the current working directory -- `~/src/yk/` --
and replicate them to the directory `~/yk` on the host `ssh.example.com` using
`rsync -az --delete-during`. If/when synchronisation completes successfully,
`rsync_cmd` will then run `ssh ssh.example.com "cd yk && cargo test"`.

To see which commands are executed, specify `-v`. For a dry-run that doesn't
run `rsync` or `ssh`, specify `-d`.

To pass additional arguments to `rsync`, put them between a pair of `--`
arguments.

`rsync_cmd` adds `dir-merge,-n /.gitignore` to the `rsync` command which does
require trusting the sender, though in `rsync_cmd`'s mirroring mode, it is
unclear whether one does need to trust the sender. `rsync_cmd` also adds
`--exclude-from=$HOME/.config/git/gitignore_global` to the `rsync` command if a
file exists at that path.


# `.rsync_cmd` configuration file

You can optionally use a `.rsync_cmd` configuration file in a directory: each
argument must be on a single line, and those arguments are prepended to the
command-line arguments. So for example:

```
$ cat .rsync_cmd
cat: .rsync_cmd: No such file or directory
$ rsync -v ssh.example.com cargo test
rsync -az --partial --info progress2 --delete-during --filter ":- .gitignore" --exclude-from /home/ltratt/.config/git/gitignore_global . ssh.example.com:yk
ssh -t ssh.example.com cd yk && cargo test
$ echo --\n--exclude=.git\n-- > .rsync_cmd
$ cat .rsync_cmd
--
--exclude=".git"
--
$ rsync -v ssh.example.com cargo test
rsync -az --partial --info progress2 --delete-during --filter ":- .gitignore" --exclude-from /home/ltratt/.config/git/gitignore_global --exclude=.git . ssh.example.com:yk
ssh -t ssh.example.com cd yk && cargo test
```

If you wish to temporarily stop the contents of `.rsync_cmd` being prepended to
the `rsync` command, specify `--ignore-conf-file` to `rsync_cmd`.
