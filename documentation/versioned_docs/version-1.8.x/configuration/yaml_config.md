---
sidebar_position: 4
---

# Configuration YAML structure

This guide explains the structure of a YAML config file. 

## Config YAML structure

Every configuration needs a YAML file containing required and optional content.

A minimum viable config should look like this:


```yaml
config:
    - {config name} : "{path of config json template}"

{config name}:
    - name: "{a unique name}"
```


Example: in `projects/infrastructure/alerting-profile/profiles.yaml`


```yaml
config:
  - profile: "projects/infrastructure/alerting-profile/profile.json"

profile:
  - name: "profile-name"
[...]
```

Any defined `- {key} : {value}` for a configuration represents a variable that can then be used in a [JSON template](configuration_structure.md#config-json-templates) by referencing
the `{key}`. This will be resolved and the `{value}` inserted into the JSON before deploying to Dynatrace.

Example: `projects/infrastructure/alerting-profile/profiles.yaml` defines a `name`, which is then used in `projects/infrastructure/alerting-profile/profile.json` as `{{.name}}`.


```yaml
[...]
profile:
  - name: "EXAMPLE Infrastructure"
[...]
```

### Name Variable
**Every config needs to provide a `name` for unique identification.** 

**Omitting the `name` variable or using a duplicate name causes a validation / deployment error.**

The `name` is used to identify configurations on a Dynatrace environment and ensure that they are updated when they already exist. 

For this, the `name` needs to be used in the [JSON template](configuration_structure.md#config-json-templates) to fill the specific name property of the configuration. 
Usually this is also just `name`, but for some configurations this may differ - please see the special cases described for [JSON templates](configuration_structure.md#config-json-templates) and refer to the [Dynatrace API documentation](https://www.dynatrace.com/support/help/dynatrace-api) if in doubt.

> When [downloading](/commands/downloading-configuration.md) names will be automatically extracted and placed in the YAML for you!

When referencing the `name` in a JSON Template it needs to be used as is, with no additional characters around it. See the single correct sample in the table below:

| Name property in JSON            | Correct |
|----------------------------------|---------|
| `"{{ .name }}"`                  | ✅       |
| `" {{ .name }}"`                 | ❌      |
| `"{{ .name }} "`                 | ❌      |
| `"Some extra text: {{ .name }}"` | ❌      |


> Should you encounter issues of configurations not being created several times instead of updated,
> check that your reference to the name does not contain any accidental spaces or other characters making
> what is sent to Dynatrace in the JSON different from the name defined in the YAML!


### Skip configuration deployment

To skip the deployment of a configuration, use the predefined `skipDeployment` parameter.

```yaml
my-config:
  - name: "My config"
  - skipDeployment: "true"
```

If you wan to enable it by default, but skip for environment or group, do the following:

```yaml
my-config:
  - name: "My config"
  - skipDeployment: "true"

my-config.development:
  - skipDeployment: "false"
```

If you want to disable it by default and enable only for environment or group: 

```yaml
my-config:
  - name: "My config"
  - skipDeployment: "false"

my-config.environment:
  - skipDeployment: "true"
```


### Specific configuration per environment or group

Configurations can be overwritten or extended:

* per environment, by adding `.{Environment}` configurations
* per group, by adding `.{GROUP}` configurations

`projects/infrastructure/notification/notifications.yaml` defines different recipients for email notifications for each environment via


```yaml
email:
    [...]

email.group:
    [...]

email.environment1:
    [...]

email.environment2:
    [...]

email.environment3:
    [...]
```


Anything in the base `email` configuration is still applied, unless it's re-defined in the `.{GROUP}` or `.{Environment}` config.

> :warning: If both environment and group configurations are defined, then environment is preferred over the group configuration.


### Referencing other configurations

In many cases, one auto-deployed Dynatrace configuration depends on another one. E.g., where most configurations depend on the management-zone defined in `projects/infrastructure/management-zone`

The tool allows your configuration to reference either the `name` or `id` of the Dynatrace object of another configuration created on the cluster.

To reference these, the dependent `config yaml` can configure a variable of the format


```
{var} : "{name of the referenced configuration}.[id|name]"
```


e.g. `projects/project-name/dashboard/dashboard.yaml` references the management-zone defined by `/projects/infrastructure/management-zone/zone.json` via

```yaml
  - managementZoneId: "projects/infrastructure/management-zone/zone.id"
```


### Referencing other JSON templates
JSON templates are usually defined inside of a project configuration and then referenced in the same project:

**testproject/auto-tag/auto-tag.yaml:**

```yaml
config:
  - application-tagging-multiproject: "application-tagging.json"

application-tagging-multiproject:
  - name: "Test Application Multiproject"
```


In this example, `application-tagging.json` is located in the `auto-tag` folder of the same project and the path to it
can be defined relative to `auto-tag.yaml` file. But, what if you would like to reuse one template defined outside of this project?
In this case, you need to define the full path of a json template:

**testproject/auto-tag/auto-tag.yaml:**

```yaml
config:
  - application-tagging-multiproject: "/path/to/project/auto-tag/application-tagging.json"

application-tagging-multiproject:
  - name: "Test Application Multiproject"
```

This would save us from content duplication and redefining the same templates over and over again.

Of course, it is also possible to reuse one template multiple times within one or different YAML file(s):
**testproject/auto-tag/auto-tag.yaml:**

```yaml
config:
  - application-tagging-multiproject: "/path/to/project/auto-tag/application-tagging.json"
  - application-tagging-tesproject: "/path/to/project/auto-tag/application-tagging.json"
  - application-tagging-otherproject: "/path/to/project/auto-tag/application-tagging.json"

application-tagging-multiproject:
  - name: "Test Application Multiproject"
  - param: "Multiproject parameter"

application-tagging-tesproject:
  - name: "Test Application Tesproject"
  - param: "Tesproject parameter"

application-tagging-otherproject:
  - name: "Test Application Otherproject"
  - param: "Otherproject parameter"
```


### Templating of environment variables

In addition to the templating of JSON files, where you need to specify the values in the corresponding YAML files, its also possible to resolve
environment variables. This can be done in any JSON or YAML file using this syntax: `{{.Env.ENV_VAR}}`.

E.g., to resolve the URL of an environment, use the following snippet:


```yaml
development:
    - name: "Dev"
    - env-url: "{{ .Env.DEV_URL }}"
    - env-token-name: "DEV_TOKEN_ENV_VAR"
```


An environment variable can also be resolved directly in the JSON. See the following example which sets the value
of an alerting profile from the env var `ALERTING_PROFILE_VALUE`.


```json
{
  "name": "{{ .name }}",
  "rules": [
    {
      "type": "APPLICATION",
      "enabled": true,
      "valueFormat": null,
      "propagationTypes": [],
      "conditions": [
        {
          "key": {
            "attribute": "WEB_APPLICATION_NAME"
          },
          "comparisonInfo": {
            "type": "STRING",
            "operator": "CONTAINS",
            "value": "{{ .Env.ALERTING_PROFILE_VALUE }}",
            "negate": false,
            "caseSensitive": true
          }
        }
      ]
    }
  ]
}
```

> :warning: Values you pass into a configuration as environment variables must not contain the `=` character.

### Newlines in variables

All line breaks in variables are escaped.

```yaml
development:
    - name: "Dev"
    - example1: "This is \\n already escaped" 
    - example2: "This will \n be escaped"
    - text: |
        This will also
        be escaped
```

### Double-Quotes in variables

Sometimes you might want to use quotes in a configuration.

To make this work in strings, you need to enclose the value in single-quotes (`'`) and escape any double-quotes (`\"`).

If you don't do that, the fully templated configurations will not be valid JSON.

```yaml
escaped_quoted_sample:
    - name: 'An Alerting Profile for \"Company Service\"'
```
