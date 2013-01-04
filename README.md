# spur.py: Run commands and manipulate files locally or over SSH using the same interface

To run echo locally:

```python
import spur

shell = spur.LocalShell()
result = shell.run(["echo", "-n", "hello"])
print result.output # prints hello
```

Executing the same command over SSH uses the same interface -- the only difference
is how the shell is created:

```python
import spur

shell = spur.SshShell(hostname="localhost", username="bob", password="password1")
result = shell.run(["echo", "-n", "hello"])
print result.output # prints hello
```

## Shell constructors

### LocalShell

Takes no arguments:

```python
spur.LocalShell()
```

### SshShell

Requires a hostname and a username. Also requires some combination of a password
and private key, as necessary to authenticate:

```python
# Use a password
spur.SshShell(
    hostname="localhost",
    username="bob",
    password="password1"
)
# Use a private key
spur.SshShell(
    hostname="localhost",
    username="bob",
    private_key_file="path/to/private.key"
)
# Use a port other than 22
spur.SshShell(
    hostname="localhost",
    port=50022,
    username="bob",
    password="password1"
)
```

## Operations

### run(command, cwd, update_env, allow_error, stdout, stderr)

Run a command and wait for it to complete. The command is expected to be a list
of strings. Returns an instance of `ExecutionResult`.

```python
result = shell.run(["echo", "-n", "hello"])
print result.output # prints hello
```

Note that arguments are passed without any shell expansion. For instance,
`shell.run(["echo", "$PATH"])` will print the literal string `$PATH` rather
than the value of the environment variable `$PATH`.

Optional arguments:

* `cwd` -- change the current directory to this value before executing the
  command
* `update_env` -- a `dict` containing environment variables to be set before
  running the command. If there's an existing environment variable with the same
  name, it will be overwritten. Otherwise, it is unchanged.
* `allow_error` -- `False` by default. If `False`, an exception is raised if
  the return code of the command is anything but 0. If `True`, a result is
  returned irrespective of return code.
* `stdout` -- if not `None`, anything the command prints to standard output
  during its execution will also be written to `stdout` using `stdout.write`.
* `stderr` -- if not `None`, anything the command prints to standard error
  during its execution will also be written to `stderr` using `stderr.write`.

## Classes

### ExecutionResult

`ExecutionResult` has the following properties:

* `return_code` -- the return code of the command
* `output` -- a string containing the result of capturing stdout
* `stderr_output` -- a string containing the result of capturing stdout

It also has the following methods:

* `to_error()` -- return the corresponding RunProcessError. This is useful if
  you want to conditionally raise RunProcessError, for instance:
  
```python
result = shell.run(["some-command"], allow_error=True)
if result.return_code > 4:
    raise result.to_error()
```

### RunProcessError

A subclass of `RuntimeError` with the same properties as `ExecutionResult`:

* `return_code` -- the return code of the command
* `output` -- a string containing the result of capturing stdout
* `stderr_output` -- a string containing the result of capturing stdout
