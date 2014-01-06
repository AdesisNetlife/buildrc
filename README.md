# .buildrc

> A generic, unopiniotated, cross-platform, language/CI-agnostic build configuration file with own DSL

**hey! do not take this seriously, work in progress**

## About

`.buildrc` aims to be a general purpose build configuration file who can be used from any 
continuous integration server

## Example

A complete `buildrc` example file, using the `buildspec` language
**Under designing process**

```ruby
# this is an in-line comment

before: 
  script:
    wget http://download.redis.io/redis-stable.tar.gz
    tar xvzf redis-stable.tar.gz
    cd redis-stable
    make
  end
end

requirements:
  satisfies: make
  satisfies: redis
  satisfies: nodejs@~0.10.0
end

env:
  set: PATH /opt/node/bin:${PATH}
  unset: NODE_PATH
end

task: test
  before: 
    run: npm install
  end

  after:
    copy: build/ ../build-$[date('Ymd-hms')]
    run: npm clean
  end

  workdir: ${TEMP}/build

  timeout:
    task: 1000
    global: 10000
  end

  run: mongo 
  run: node app.js 

  events:
    error:
      run: echo 'Cannot run the task (${CODE}): ${ERROR}' >> error.log
    end
    success:
      run: pidof node | kill -SIGTERM
    end
    timeout:
      run: echo 'Timeout exceded for ${TASK}' >> error.log
    end
  end
end
```

## Buildspec language

Buildspec is a minimalist DSL created to be used in `buildrc` configuration files

### Configuration blocks

#### requirements
Define pre-execution requirements

#### task
Type: `block`

Register a callable task

#### env
Type: `set|unset` 
Scope: `global|local`

Set/unset environment variables

##### set <variable> <value>
##### unset <variable>

#### workdir
Type: `string`
Scope: `global|local`

Set a working directory

#### run
Type: `command`
Scope: `global|local`

Run a command as isolated statement with own DSL support and evaluate the exit code.
Supports pipes, redirects and standard command sentences 

#### script
Type: `string`
Scope: `global|local`

Run a bash/batch script and evaluate the exit code. 
No Buildspec expressions are supported in this configuration block

#### copy
Type: `copy block`
Scope: `local`

A file system command helper to copy files or directories

#### move
Type: `block{remove}`
Scope: `local`

A file system command helper to move files or directories

#### remove
Type: `block{remove}`
Scope: `local`

A file system command helper to remove files or directories

#### events
Type: `block`
Scope: `local`

Task state execution events configuration

#### timeout
Type: `timeout block|number` 
Scope: `global|local`

Define a max execution timeout. 
Optionally you can declare a timeout block with a per-task-specific timeout or a global timeout

##### task <miliseconds>
##### global <miliseconds>

### Expressions

#### Environemnt variables
In order to provide cross-platform support for environment variables in builrc,
it should be declared using the following notation:

```
${VARIABLE_NAME}
```

#### Build-in variables

```
$[variable/command]
```

### Execution priority

1. `workdir`
2. `before`
3. `env`
4. `requirements`
5. `task`
6. `after`
7. `events`

## Wishful thinking

- Specific logic operators
- Standard input/output redirection with own DSL operators
- OS-agnostic command helpers (HTTP client, network testing, fs utilities, unix-like binaries emulation...)

## License

Copyright (c) Adesis Netlife S.L and contributors

Specification licensed under Creative Commons CC-BY-SA
