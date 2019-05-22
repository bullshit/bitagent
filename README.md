bitagent
===

bitagent is a small service to help you share secrets between processes. Run
locally, each instance of bitagent listens on a Unix socket and is able to store
one secret. Instead of storing a session key or password in a file, store it in
a bitagent process. This keeps it off your disk, out of `ps` output, and
eliminates accidental password leaks to your shell's history file.

Why
---

Occasionally there are passwords or session keys that need to be accessed from
multiple processes and login sessions but that should not be written to disk.
The inspiration for this is Bitwarden's CLI, which requires a session key to be
passed or set in the environment. Sharing this session key between shell
sessions (or when using
[ansible](https://github.com/c0sco/ansible-modules-bitwarden)) can be
cumbersome.  

Each bitagent process is capable of storing only one secret. This keeps the code
simple, which helps keep it performant and reduces the chance of errors.

Installation & usage
---

To install bitagent, use the standard `go install` process.

```bash
go install github.com/mjslabs/bitagent
```

Launch bitagent using your system's preferred method of backgrounding a process,
e.g.

```bash
${GOBIN}/bitagent & disown
```

By default bitagent will create `~/.bitagent.sock` for communication. You can
specify an alternative location for the socket by passing it as the one argument
to bitagent.  

To store a secret, send a `P` command. Here's an example using netcat.

```bash
echo "Pmysecret" | nc -U ~/.bitagent.sock -N
```

To retreive the secret, use `G`.

```bash
echo "G" | nc -U ~/.bitagent.sock -N
```

The easiest way to work with bitagent is by making a wrapper script for your
use case. See [examples](examples), which includes such a script for use with
the Bitwarden CLI.

Caveats
---

bitagent uses [memguard](https://github.com/awnumar/memguard), which attempts to
stop the part of bitagent's memory that is holding a secret from being paged
out or included in core dumps. This has not been fully vetted by the authors of
bitagent.  

bitagent defaults to storing up to a 256 byte secret. This is tunable at the
top of [main.go](main.go).  

The only thing stopping someone from accessing your secret in bitagent is the
permissions on the socket file. These default to a sane value, but there are
no guarantees that this is the best practice for all environments. You should
only run bitagent on trusted machines. This is a similar to how you would treat
your SSH private key file.
