# shdotenv

dotenv support for shell and POSIX-compliant `.env` syntax specification.

![GitHub Workflow Status](https://img.shields.io/github/workflow/status/ko1nksm/shdotenv/macOS?logo=github)

Quoting [bkeepers/dotenv][dotenv]:

> Storing [configuration in the environment](http://12factor.net/config) is one of the tenets of a [twelve-factor app](http://12factor.net). Anything that is likely to change between deployment environments–such as resource handles for databases or credentials for external services–should be extracted from the code into environment variables.

[dotenv]: https://github.com/bkeepers/dotenv

## The goals of this project

1. Provide language-independent CLI utilities
2. Provide a library that can safely load .env file from shell scripts
3. Define POSIX shell compatible .env syntax specification
4. Support for .env syntax dialects for interoperation

## Requirements

`shdotenv` is a single file shell script with embedded awk script.

- POSIX shell (dash, bash, ksh, zsh, etc)
- awk (gawk, nawk, mawk, busybox awk)

## Install

Download `shdotenv` (shell script) from [releases](https://github.com/ko1nksm/shdotenv/releases).

```console
$ wget https://github.com/ko1nksm/shdotenv/releases/latest/download/shdotenv -O $HOME/bin/shdotenv
$ chmod +x $HOME/bin/shdotenv
```

### Build your own

Requires [shfmt](https://github.com/mvdan/sh).

```console
$ git clone https://github.com/ko1nksm/shdotenv.git
$ cd shdotenv
$ make
$ make install PREFIX=$HOME
```

## How to use

### Usage

```
Usage: shdotenv [OPTION]... [--] [COMMAND [ARG]...]

  -d, --dialect DIALECT  Specify the .env dialect [default: posix]
                           (posix, ruby, node, python, php, go, rust, docker)
  -s, --shell SHELL      Output in the specified shell format [default: posix]
                           (posix, fish)
  -e, --env ENV_PATH     Location of the .env file [default: .env]
                           Multiple -e options are allowed
  -o, --overload         Overload predefined environment variables
  -n, --noexport         Do not export keys without export prefix
  -g, --grep PATTERN     Output only those that match the regexp pattern
  -k, --keyonly          Output only variable names
  -q, --quiet            Suppress all output
  -v, --version          Show the version and exit
  -h, --help             Show this message and exit
```

### Use as CLI utility

Set environment variables and execute the specified command.

```sh
shdotenv [OPTION]... <COMMAND> [ARGUMENTS]...
```

### Use as shell script library

Load the .env file into the shell script.

```sh
eval "$(shdotenv [OPTION]...)"
```

When run on the shell, it exports the environment variables to the current shell.

### Additional CLI utility

#### contrib/dockerenv

The `docker` command has the `--env-file` option, but it only supports setting simple values.

- [docker cannot pass newlines from variables in --env-file files](https://github.com/moby/moby/issues/12997)

This tool makes the files read by `--env-file` compatible with the `.env` format, and supports variable expansion and newlines.

Example: (Use `dockerenv` instead of `docker`)

```sh
dockerenv run --env-file .env -it debian
```

## .env file syntax

```sh
# dotenv posix
# This line is a comment, The above line is a directive
COMMENT=This-#-is-a-character # This is a comment

UNQUOTED=value1 # Spaces and some special characters cannot be used
SINGLE_QUOTED='value 2' # Cannot use single quote
DOUBLE_QUOTED="value 3" # Some special characters need to be escaped

MULTILINE="line1
line2: \n is not a newline
line3"
LONGLINE="https://github.com/ko1nksm\
/shdotenv/blob/main/README.md"

ENDPOINT="http://${HOST}/api" # Variable expansion requires braces

export EXPORT1="value"
export EXPORT2 # Equivalent to: export EXPORT2="${EXPORT2:-}"
```

- The first line is a directive to distinguish between .env syntax dialects
- Comments at the end of a line need to be preceded by spaces before the `#`
- Spaces before and after `=` are not allowed
- The special characters allowed in unquoted value are `#` `%` `+` `,` `-` `.` `/` `:` `=` `@` `^` `_`
- Single-quoted values cannot contains single quote in it
- The characters `$` <code>\`</code> `"` `\` in the double-quoted value must be escaped with `\`
- No support for backslash escapes except for the above (i.e., `\n` is not a newline)
- Variable expansion is only available if it is enclosed in double quotes
- Bracing is required for variable expansion (Only `${VAR}` is supported)
- The `\` at the end of a line in double quoted value means line continuation

Detailed [POSIX-compliant .env syntax specification](docs/specification.md)

### Directive

Specifies the dotenv syntanx dialect that this `.env` file.

```sh
# dotenv <DIALECT>
```

Example:

```sh
# dotenv ruby
```

### Supported dialects

The formal `.env` syntax for this project is `posix` only.
The `posix` is a subset of the POSIX shell and is compatible with shell scripts.
Support for other .env syntax dialects is for interoperability purposes.
Compatibility will be improved gradually, but is not fully compatible.
Reports of problems are welcome.

- docker: [docker](https://docs.docker.com/engine/reference/commandline/run/#set-environment-variables--e---env---env-file)
- ruby: [dotenv](https://github.com/bkeepers/dotenv)
- node: [dotenv](https://github.com/motdotla/dotenv) + [dotenv-expand](https://github.com/motdotla/dotenv-expand)
- python: [python-dotenv](https://github.com/theskumar/python-dotenv)
- php: [phpdotenv](https://github.com/vlucas/phpdotenv)
- go: [godotenv](https://github.com/joho/godotenv)
- rust: [dotenv](https://github.com/dotenv-rs/dotenv)

[Comparing Dialects](docs/dialects.md)

## .shdotenv

Specifies options for shdotenv. Currently, only `dialect` is supported.
It is recommended that the dotenv dialect be specified with the `dotenv` directive.
The `.shdotenv` setting is for personal use in projects where it is not allowed.

```
dialect: <DIALECT>
```

Example:

```
dialect: ruby
```

## Environment Variables

| name           | description                              | default |
| -------------- | ---------------------------------------- | ------- |
| SHDOTENV_SHELL | Shell format to output (`posix`, `fish`) | `posix` |
| SHDOTENV_AWK   | Path of the `awk` command                | `awk`   |
