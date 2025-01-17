# Dockerfile Validation

The validator is intended to perform the same kinds of validation that the Docker builder performs when building an image.
Any errors that are generated by the validator should also be generated by the Docker builder (albeit the message strings itself may differ).
If the validator generates an error for a Dockerfile that the Docker builder is able to build, that is considered to be a bug.

However, errors that the Docker builder generates may not necessarily be replicated by the validator due to difficulties in verifying the validity of a given Dockerfile instruction and its arguments given the builder's build context and other factors.

The currently supported version of the Docker builder is **Docker CE 17.09 [2017-09-26]**.

## Command Line Interface

If no file is specified, the CLI will attempt to validate the contents of a file named `Dockerfile` in the current working directory if it exists.

### Help

```batch
> dockerfile-utils lint --help
Usage: dockerfile-utils lint [options] [file]

Options:

  -h, --help                Output usage information
  -j, --json                Output in JSON format
```

### Example

```Dockerfile
FROM node
HEALTHCHECK --interva=30s CMD ls
RUN "echo" ls \
"echoS"sdfdf \
asdfasdf
copy . .
ADD app.zip
CMD ls
```

#### CLI Output

```b
> dockerfile-utils lint
Line: 2
HEALTHCHECK --interva=30s CMD ls
            ^^^^^^^^^
Error: Unknown flag: interva

Line: 4
Warning: Empty continuation line

Line: 7
  copy . .
  ^^^^
Warning: Instructions should be written in uppercase letters

Line: 8
ADD app.zip
    ^^^^^^^
Error: ADD requires at least two arguments
```

#### JSON Output

For readability purposes, the output below has been formatted manually.
The output on the command line will not include any whitespaces.

```b
> dockerfile-utils lint -j
[
  {
    "range": {
      "start": { "line": 1, "character": 12 },
      "end": { "line": 1, "character": 21 }
    },
    "message": "Unknown flag: interva",
    "severity": "error"
  },
  {
    "range": {
      "start": { "line": 3, "character": 0 },
      "end": { "line": 4, "character": 0 }
    },
    "message": "Empty continuation line",
    "severity": "warning"
  },
  {
    "range": {
      "start": { "line": 6, "character": 2 },
      "end": { "line": 6, "character": 6 }
    },
    "message": "Instructions should be written in uppercase letters",
    "severity": "warning"
  },
  {
    "range": {
      "start": { "line": 7, "character": 4 },
      "end": { "line": 7, "character": 11 }
    },
    "message": "ADD requires at least two arguments",
    "severity": "error"
  }
]
```

## Supported Validation Checks

### General

#### Instructions

- instructions should be written in uppercase
- instruction has no arguments
- instruction has an insufficient number of arguments
- unknown instruction detected
- duplicate instruction flags detected
- unknown instruction flag detected
- instruction flag has no value defined

#### Directives

- invalid value specified for `escape` parser directive
- directives should be written in lowercase.

#### Others

- empty continuation lines

### CMD

- multiple `CMD` instructions detected

### ENTRYPOINT

- multiple `ENTRYPOINT` instructions detected

### ENV

- syntax missing equals sign '`=`'
- syntax missing single quote '`'`'
- syntax missing double quotes '`"`'
- property has no name

### EXPOSE

- invalid container port specified
- invalid protocol specified

### FROM

- `FROM` instruction not found at the beginning of the Dockerfile
- invalid build stage name specified
- duplicate build stage name detected
- second argument detected but not an `AS`

### HEALTHCHECK

- `CMD` form has no arguments
- `NONE` form has arguments defined
- type that is not `CMD` or `NONE` detected
- `--retries` flag has invalid syntax
- `--retries` value is not at least one
- duration of `--interval`, `--start-period`, or `--timeout` is invalid
- duration of `--interval`, `--start-period`, or `--timeout` is less than one millisecond
- duration of `--interval`, `--start-period`, or `--timeout` has an unknown unit of time specified
- multiple `HEALTHCHECK` instructions detected

### LABEL

- syntax missing equals sign '`=`'
- syntax missing single quote '`'`'
- syntax missing double quotes '`"`'
- property has no name

### MAINTAINER

- use of deprecated instruction detected

### ONBUILD

- can't chain `ONBUILD` instruction with `ONBUILD ONBUILD`
- invalid `ONBUILD` trigger instruction

### SHELL

- `SHELL` not written in JSON form

### STOPSIGNAL

- invalid stop signal
