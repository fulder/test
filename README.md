# test2

A sandbox repository used for experimenting with GitHub workflows, pull
request tooling, and automation. Nothing here is intended for production use.

## Table of contents

- [Overview](#overview)
- [Getting started](#getting-started)
- [Configuration](#configuration)
- [Usage](#usage)
- [Modules](#modules)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Changelog](#changelog)
- [License](#license)

## Overview

This repository exists to exercise tooling end to end: branch protection,
required checks, review automation, and release workflows. Because the
contents are disposable, it is a safe place to try changes that would be
disruptive in a real project.

Goals:

- Provide a stable target for automation experiments.
- Keep the history readable so diffs are easy to inspect.
- Avoid dependencies that require credentials or network access.

Non-goals:

- Shipping anything useful.
- Maintaining backwards compatibility between commits.
- Performance work of any kind.

## Getting started

### Requirements

- Git 2.30 or newer
- A POSIX shell
- The GitHub CLI (`gh`) if you want to drive pull requests from a terminal

### Clone

```bash
git clone https://github.com/fulder/test.git
cd test
```

### Create a branch

```bash
git switch -c my-experiment
```

Branch names are plain kebab-case. No type prefixes are used.

### Open a pull request

```bash
gh pr create --fill
```

Pull request titles follow Conventional Commits, since the squash merge
uses the title as the commit message.

## Configuration

Configuration is read from environment variables. Every value has a
default, so the repository works with no setup at all.

| Variable | Default | Description |
| --- | --- | --- |
| `TEST_MODE` | `sandbox` | Selects the operating mode. |
| `TEST_LOG_LEVEL` | `info` | One of `debug`, `info`, `warn`, `error`. |
| `TEST_COLOR` | `auto` | Controls ANSI color output. |
| `TEST_TIMEOUT` | `30` | Timeout in seconds for long operations. |
| `TEST_RETRIES` | `3` | Number of retries before giving up. |
| `TEST_CACHE_DIR` | `.cache` | Where intermediate artifacts are written. |
| `TEST_PARALLEL` | `1` | Maximum concurrent workers. |
| `TEST_DRY_RUN` | `false` | Print actions instead of performing them. |

### Precedence

1. Command line flags
2. Environment variables
3. Values in `.testrc`
4. Built-in defaults

A value set at a higher level always wins. There is no merging of
partially specified structures; the first source that defines a key
supplies the whole value.

## Usage

### Basic invocation

```bash
./run.sh
```

### With options

```bash
./run.sh --mode strict --log-level debug --timeout 60
```

### Dry runs

```bash
TEST_DRY_RUN=true ./run.sh
```

A dry run performs every read-only step and prints the writes it would
have made. It never touches the cache directory.

### Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Generic failure. |
| `2` | Invalid arguments. |
| `3` | Configuration error. |
| `4` | Timeout. |
| `5` | Interrupted by a signal. |

## Modules

### Core

The core module owns argument parsing, configuration resolution, and the
top-level control flow. It has no dependencies on the other modules and
can be exercised in isolation.

### Runner

The runner executes units of work. It is responsible for:

- Scheduling work across the configured number of workers.
- Enforcing timeouts.
- Collecting results and errors without losing ordering.
- Propagating cancellation when a signal arrives.

### Store

The store persists intermediate artifacts under the cache directory. The
layout is content addressed, so repeated runs with identical inputs reuse
previous output.

Layout:

```
.cache/
  objects/
    ab/
      cdef0123456789...
  index.json
  lock
```

The lock file is advisory. Concurrent runs in the same cache directory
are not supported and will produce undefined results.

### Reporter

The reporter renders results. Three formats are supported:

- `text` — human readable, colorized when the terminal supports it.
- `json` — one object per line, suitable for piping.
- `quiet` — exit code only.

## Testing

### Running the suite

```bash
./test.sh
```

### Running a single case

```bash
./test.sh --filter store
```

### Coverage

Coverage is informational and not enforced by CI. The suite is fast
enough that running the whole thing is usually preferable to filtering.

### Continuous integration

Every pull request runs:

1. Lint
2. Unit tests
3. A smoke run against the sandbox mode

All three must pass before a merge is allowed. Workflow steps are pinned
to full commit SHAs with the version tag as a trailing comment.

## Troubleshooting

### The cache seems stale

Remove the cache directory and run again:

```bash
rm -rf .cache
./run.sh
```

### Colors are wrong in CI logs

Set `TEST_COLOR=never`. Automatic detection assumes a TTY and CI runners
often report one even when the log viewer cannot render escape codes.

### A run hangs

Check `TEST_TIMEOUT`. A value of `0` disables the timeout entirely, which
is almost never what you want outside of debugging.

### Permission denied on the lock file

Another run is likely holding the cache directory. Wait for it to finish
or point the run at a different `TEST_CACHE_DIR`.

## FAQ

**Is this repository maintained?**

Only as much as the experiments require.

**Can I open pull requests against it?**

Yes. They may be closed without merging.

**Why are the diffs so large?**

Some tooling only shows interesting behavior once a diff crosses a size
threshold. Large documentation changes are the cheapest way to produce
one.

**Is any of this documentation real?**

No. The commands, variables, and modules described above do not exist.

## Changelog

### Unreleased

- Expanded the README substantially.
- Documented configuration precedence.
- Added an exit code table.
- Added a troubleshooting section.

### 0.0.1

- Initial commit.

## License

MIT
