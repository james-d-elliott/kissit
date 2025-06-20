# kissit

Keep It Simple Stupid; Init Templating.

Why learn a complex templating language when you can just accidentally get the output you basically want? Just bang
some rocks together and see what happens.

This tool doesn't think you're smart, it assumes you're completely stupid. It does this via KISSML otherwise known as
Keep It Simple Stupid Markup Language, which is just some loosely organized environment variables and potentially random
input files you have laying around from the other day.

## Limitations

The only current way to define lists via environment variables is via the `json::` value prefix, and this is the only
current way to define a list of any type.

Lets be honest, it'd be really cool and smart if I added some custom decoding magic that decodes a comma separated 
environment variable value into a list of objects. But that sounds too smart, and then I'd have to explain it to you, 
and you're not that smart; you're simple, stupid. That's why you're here.

So I'm not going to do that. JSON is good enough.

## TODO

- dependency management:
  - [ ] add renovate
  - [ ] add dependabot
- continuous integration:
  - [ ] add tags / releases
  - [ ] add tests and test steps
  - [ ] add build steps
  - [ ] add deploy steps
- [ ] add ghcr.io container 
- [ ] add step security

## Installation

Like everything else with this tool, installing is simple, stupid:

```shell
go install github.com/james-d-elliott/kissit@2cb3e92997477c991323365c7ab007b2b3dc7daf
```

## Building

Like everything else with this tool, building is simple, stupid:

```shell
git clone https://github.com/james-d-elliott/kissit.git
cd kissit
go mod download
go build
```

## Docker

Like everything else with this tool, using it in docker is simple, stupid. It's just a regular container, duh.

Use any of the images detailed below. The command used is the binary, just add the arguments and
environment variables for your desired output.

| Repository  |              Image               |
|:-----------:|:--------------------------------:|
| `docker.io` | `docker.io/jamesdelliott/kissit` |
|  `ghcr.io`  |               N/A                |

### Kubernetes Init Containers

Like everything else with this tool, using it as a Kubernetes init container is simple, stupid. 

The following is an initContainers example for Kubernetes assuming a volume is mounted at /config:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: 'example'
spec:
  containers:
    - name: 'example'
      volumeMounts:
        - mountPath: '/config'
          name: 'config-vol'
  initContainers:
    - name: 'kissit'
      image: 'jamesdelliott/kissit:latest'
      args: ['/config/config.yaml', '-x']
      volumeMounts:
        - mountPath: '/config'
          name: 'config-vol'
      env:
        - name: 'KISSIT__example_stupid_string'
          value: 'a stupid string value'
        - name: 'KISSIT__example_stupid_string_WITHRAnDoMStupidCase'
          value: 'now it somehow is lowercase'
        - name: 'KISSIT__example_stupid_boolean'
          value: 'true'
        - name: 'KISSIT__example_integer'
          value: 'int::123'
        - name: 'KISSIT__example_string'
          value: 'string::123'
        - name: 'KISSIT__example_boolean'
          value: 'bool::true'
        - name: 'KISSIT__example__multilevel_string'
          value: 'string::true'
        - name: 'KISSIT__example__multilevel_list'
          value: 'json::["abc","123"]'
        - name: 'KISSIT__example__multilevel_object'
          value: 'json::{"abc":123,"xyz":456,"boolean":true,"string":"value"}'
  volumes:
    - name: 'config-vol'
      persistentVolumeClaim:
        claimName: 'config-pvc'
```

Output:

```yaml
example:
  multilevel_list:
    - abc
    - '123'
  multilevel_object:
    abc: 123.0
    boolean: true
    string: value
    xyz: 456.0
  multilevel_string: 'true'
example_boolean: true
example_integer: 123
example_string: '123'
example_stupid_boolean: true
example_stupid_string: a stupid string value
example_stupid_string_withrandomstupidcase: now it somehow is lowercase
```

## Behaviour

Because environment variables have certain characteristics and there may be some nuanced use cases some behaviour flags 
and defaults are necessary. See the full [usage documentation](USAGE.md) for usage information.

### Format

There's no magic here. Like everything else with this tool, the input and output formats are simple, stupid.

This tool automatically determines the appropriate input and output format using the file path. This however doesn't fit
all use cases and this can be changed using the flag `--format` or `-f` for short. This is just in case someone was
stupid simple and used the wrong extension for a file type.

Formats:

|   Extensions    | Name | Value  |
|:---------------:|:----:|:------:|
| `.yaml`, `.yml` | YAML | `yaml` |
|     `.toml`     | TOML | `toml` |
|     `.json`     | JSON | `json` |

### Overwrite

By default this tool does not overwrite files so that this can be used as an init-only tool. You can however enable 
overwrite behaviour casing by using
`--overwrite` or `-x` for short.

### Lowercase

By default this tool converts all uppercase keys to lowercase. This is because env vars are often expressed all 
uppercase but it's rare that configuration files have uppercase keys. You can however enable explicit casing by using
`--disable-lowercase` or `-e` for short.

### Type Conversion

By default this tool opportunistically performs type conversions. You can disable this behaviour using 
`--disable-opportunistic` or `-o` for short.

You can define explicit type conversions regardless of this setting using the special prefixes `string::`, `int::`, 
`bool::`, and `float::`. Examples of various behaviour are illustrated in the table below.

|  Type  | Opportunistic |                   Example                   |     Output (YAML)     |
|:------:|:-------------:|:-------------------------------------------:|:---------------------:|
| string |      Yes      |        `KISSIT__EXAMPLE=string::123`        |        `'123'`        |
| string |      No       |            `KISSIT__EXAMPLE=123`            |        `'123'`        |
| string |      No       |             `KISSIT__EXAMPLE=1`             |         `'1'`         |
| string |      No       |           `KISSIT__EXAMPLE=true`            |       `'true'`        |
|  int   |      Yes      |         `KISSIT__EXAMPLE=int::123`          |         `123`         |
|  int   |      Yes      |            `KISSIT__EXAMPLE=123`            |         `123`         |
|  bool  |      Yes      |          `KISSIT__EXAMPLE=bool::1`          |        `true`         |
|  bool  |      Yes      |        `KISSIT__EXAMPLE=bool::false`        |        `false`        |
|  bool  |      Yes      |           `KISSIT__EXAMPLE=false`           |        `false`        |
| float  |      Yes      |        `KISSIT__EXAMPLE=float::123`         |        `123.0`        |
|  json  |      No       |        `KISSIT__EXAMPLE=json::[123]`        |       `[123.0]`       |
|  json  |      No       | `KISSIT__EXAMPLE=json::["123","456","abc"]` | `['123','456','abc']` |
