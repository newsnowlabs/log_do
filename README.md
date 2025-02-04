# log_do

log_do is a bash library to assist writing self-documenting self-logging bash scripts, by simply prefacing each command with `log_do`.

`log_do` makes bash scripts easy to develop, debug and use, through:

- logging every command, its stdout, stderr and exit code;
- providing an automagic dry-run mode;
- providing an automagic interactive execution mode.

Thanks to these features, `log_do` helps you build complex sysadmin bash scripts iteratively and keep them well maintained.

Thanks to the built-in dry-run mode, that lists all commands without executing them, `log_do` reduces the cognitive load needed for running scripts with which you are (or have become over time) unfamiliar.

## Example 1

```
# Load log_do
$ . /usr/local/bin/log_do

# Log but DON'T run 'head -n 3 /etc/os-release'
$ log_do -vv -s head -n 3 /etc/os-release
2025-01-16.00:12:36.697122|2364797| $ ls -l /var/www/ /qwe

# Log and run 'head -n 3 /etc/os-release' and log its exit code
$ log_do -vv -s -x head -n 3 /etc/os-release
2025-01-16.00:16:59.281136|2364797| $ head -n 3 /etc/os-release
2025-01-16.00:16:59.297366|2364797|   > PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
2025-01-16.00:16:59.306301|2364797|   > NAME="Debian GNU/Linux"
2025-01-16.00:16:59.312857|2364797|   > VERSION_ID="12"
2025-01-16.00:16:59.318454|2364797|   * returned 0 (from head -n 3 /etc/os-release)

# Log and interactively run 'head -n 3 /etc/os-release' and log its exit code
$ log_do -vv -s -xx head -n 3 /etc/os-release
2025-01-16.00:36:31.657803|2364797| $ head -n 3 /etc/os-release <--- Execute? (Y/n/q): Y
2025-01-16.00:36:32.815756|2364797|   > PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
2025-01-16.00:36:32.823282|2364797|   > NAME="Debian GNU/Linux"
2025-01-16.00:36:32.831343|2364797|   > VERSION_ID="12"
2025-01-16.00:36:32.841251|2364797|   * returned 0 (from head -n 3 /etc/os-release)

# Set log_do options -vv -s -x as defaults for future log_do commands
log_do_setopt -vv -s -x

# Log and run 'head -n 3 /etc/os-release' and its exit code, also enabling stdout to be piped through sort
$ log_do --stdout head -n 3 /etc/os-release | sort
2025-01-16.00:30:39.349854|2381839| $ head -n 3 /etc/os-release
2025-01-16.00:30:39.374459|2381839|   > PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
2025-01-16.00:30:39.381221|2381839|   > NAME="Debian GNU/Linux"
2025-01-16.00:30:39.388035|2381839|   > VERSION_ID="12"
2025-01-16.00:30:39.396135|2381839|   * returned 0 (from head -n 3 /etc/os-release)
NAME="Debian GNU/Linux"
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
VERSION_ID="12"

# Log and run 'head -n 3 /etc/os-release' and its exit code, enabling stdout to be piped through 'log_do sort'
$ log_do --stdout head -n 3 /etc/os-release | log_do sort
$ log_do --stdout head -n 3 /etc/os-release | log_do sort
2025-01-16.00:34:25.640307|2382075| $ sort
2025-01-16.00:34:25.641831|2382074| $ head -n 3 /etc/os-release
2025-01-16.00:34:25.666227|2382074|   > PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
2025-01-16.00:34:25.674850|2382074|   > NAME="Debian GNU/Linux"
2025-01-16.00:34:25.683730|2382074|   > VERSION_ID="12"
2025-01-16.00:34:25.691970|2382074|   * returned 0 (from head -n 3 /etc/os-release)
2025-01-16.00:34:25.703612|2382075|   > NAME="Debian GNU/Linux"
2025-01-16.00:34:25.718962|2382075|   > PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
2025-01-16.00:34:25.727639|2382075|   > VERSION_ID="12"
2025-01-16.00:34:25.735719|2382075|   * returned 0 (from sort)
```

## Example 2

Create a test script called `/tmp/test.sh`:

```
cat <<'_EOE_' >/tmp/test.sh && chmod 755 /tmp/test.sh
# Load log_do
. /usr/local/bin/log_do

# Set log_do options '-vv -s' as defaults for future log_do commands
log_do_setopt -vv -s

# Set log_do '-x' option, if this script called with '-x'
[ "$1" == "-x" ] && log_do_setopt -x

log "Welcome to my test script!"

log_do head -n 3 /etc/os-release

log_do vmstat
_EOE_
```

Run `/tmp/test.sh`:

```
$ /tmp/test.sh
2025-01-16.00:50:21.526360|2383597| Welcome to my test script!
2025-01-16.00:50:21.535548|2383597| $ head -n 3 /etc/os-release
2025-01-16.00:50:21.542995|2383597| $ vmstat
```

Run `/tmp/test.sh -x`:

```
$ /tmp/test.sh -x
2025-01-16.00:50:23.201421|2383610| Welcome to my test script!
2025-01-16.00:50:23.210891|2383610| $ head -n 3 /etc/os-release
2025-01-16.00:50:23.235229|2383610|   > PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
2025-01-16.00:50:23.242982|2383610|   > NAME="Debian GNU/Linux"
2025-01-16.00:50:23.251097|2383610|   > VERSION_ID="12"
2025-01-16.00:50:23.260123|2383610|   * returned 0 (from head -n 3 /etc/os-release)
2025-01-16.00:50:23.267192|2383610| $ vmstat
2025-01-16.00:50:23.295717|2383610|   > procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
2025-01-16.00:50:23.301974|2383610|   > r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
2025-01-16.00:50:23.307295|2383610|   > 2  0 782224 4638524  80908 4634420    0    0     0     0    0    0  0  0 100  0  0
2025-01-16.00:50:23.315525|2383610|   * returned 0 (from vmstat)
```

## Command reference

```
        log_do <command> - Optionally log and execute <command>, optionally suppressing stdout/stderr
           log <message> - Log <message>
      log_push <message> - Log <message> and indent
       log_pop <message> - Log <message> and outdent
 log_do_setopt <options> - Set default <options> to apply to future log_do commands
          log_do_running - True if execution enabled via -x or -xx
```

## log_do / log_do_setopt options

Use the following options with `log_do` or, to set defaults, with `log_do_setopt`.

```
# Execution and flow-control
                   -x: Execute command / disable dry-run
                  -xx: Execute command interactively (yes/no/quit)
                   +x: Disable command execution / enable dry-run

             --strict: Enables strict mode, which may force the script to exit on failure.
          --no-strict: Disables strict mode, potentially allowing the script to continue on failure.

# Verbosity options
                   -v: Log the command to be run
                  -vv: Log the command to be run, and exit code
                   +v: Disable command logging

# Disabling logging
      --no-log-stdout: Don't log stdout
      --no-log-stderr: Don't log stderr
   --no-log|--no-pipe: Don't log stdout or stderr

# Enabling logging
         --log|--pipe: Log stdout and stderr
         --log-stdout: Log stdout
         --log-stderr: Log stderr

# Suppresssing stdout/stderr
          --no-stdout: Suppress standard output
          --no-stderr: Suppress standard error
          --silent|-s: Suppress BOTH standard output and standard error

# Enabling stdout/stderr
--stdout|+stdout|+out: Enables standard output
--stderr|+stderr|+err: Enable standard error
       --no-silent|+s: Enable BOTH standard output and standard error (deactivate silent mode)

# Logging redirection
--output-file: Sets a specified file as the output destination for log messages.
```

## Further examples

See docker-image-deploy