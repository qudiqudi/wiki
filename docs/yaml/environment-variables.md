---
id: env-vars
title: Environment Variables
---

Environment variables may be used in your configuration YAML files to provide certain values
dynamically. This is especially useful in certain environments, such as Kubernetes. [Secrets] serve
a similar purpose, however those still require you to modify a file to provide the values.

[Secrets]: ./secrets-reference.md

:::note Version Requirement

Environment variables support was introduced in `v4.3.0`.

:::

The following rules apply:

- Environment variables may only be used in configuration YAML files.
- You may only substitute single (scalar) values in YAML; not complex, multi-line data.
- Environment variable names are case-sensitive.

## Syntax

```yml
!env_var MY_ENV_VAR my_default_value
```

Where:

- `MY_ENV_VAR` is the name of the environment variable to substitute the value of.
- `my_default_value` is an **optional** default value to use if the specified environment variable
  is either not defined, a blank string, or contains only whitespace characters.

:::caution

If an environment variable cannot be found or does not have a value as defined above, *and* no
default value is provided, an exception is thrown and configuration loading will fail for that
specific instance.

:::

### Default Value Parsing Behavior

Special notes about parsing and the default value.

- Spaces are allowed in the default value. Example:

  ```yml
  !env_var FUBAR my default value
  ```

  The default value will be `my default value`.

- If either single (`'`) or double (`"`) quotation marks surround the default value, they are
  stripped. Example:

  ```yml
  !env_var FUBAR "my default value"
  ```

  The default value will be `my default value` (without quotes).

  :::info

  Quotation marks are **not** required for default values that contain spaces!

  :::

## Example

Suppose you have the following `recyclarr.yml` file and you want to move your `base_url` and
`api_key` to environment variables:

```yml
radarr:
  radarr4k:
    base_url: http://localhost:7878
    api_key: bf99da49d0b0488ea34e4464aa63a0e5
```

Using Linux OS as an example, define some environment variables with the above API key and base URL
values:

```bash
export RADARR_BASE_URL="http://localhost:7878"
export RADARR_API_KEY="bf99da49d0b0488ea34e4464aa63a0e5"
```
Or when using docker compose, simply pass your environment variables via the service environment,
where RADARR_IP and RADARR_API_KEY are defined via a [docker compose .env file:](https://docs.docker.com/compose/environment-variables/set-environment-variables/#compose-file)

```yml
environment:
      - RADARR_BASE_URL=http://$RADARR_IP:7878
      - RADARR_API_KEY=$RADARR_API_KEY
```

Then, simply replace those properties in your configuration YAML with the environment variables
syntax:

```yml
radarr:
  radarr4k:
    base_url: !env_var RADARR_BASE_URL
    api_key: !env_var RADARR_API_KEY
```

Note that, above, no default values are specified for either environment variable. So if those end
up being empty or are deleted later, you'll get failures when loading this config.
