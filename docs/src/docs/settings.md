---
description: Meltano supports a number of settings that allow you to fine tune its behavior, which are documented here.
---

# Settings Reference

Meltano supports a number of settings that allow you to fine tune its behavior, which are documented here.
To quickly find the setting you're looking for, use the Table of Contents in the sidebar.

As described in the [Configuration guide](/docs/configuration.html#configuration-layers), Meltano will determine the values of these settings by first looking in [**the environment**](/docs/configuration.html#configuring-settings), then in your project's [**`.env` file**](/docs/project.html#env), and finally in your [**`meltano.yml` project file**](/docs/project.html#meltano-yml-project-file), falling back to a default value if nothing was found.

You can use [`meltano config meltano list`](/docs/command-line-interface.html#config) to list all available settings with their names, [environment variables](/docs/configuration.html#configuring-settings), and current values.

Configuration that is _not_ environment-specific or sensitive should be stored in your [`meltano.yml` project file](/docs/project.html#meltano-yml-project-file) and checked into version
control. Sensitive values like passwords and tokens are most appropriately stored in the environment or your project's [`.env` file](/docs/project.html#env).

[`meltano config meltano set <setting> <value>`](/docs/command-line-interface.html#config), which is used in the examples below, will automatically store configuration in `meltano.yml` or `.env` as appropriate.

If supported by the plugin type, its configuration can be tested using [`meltano config <plugin> test`](/docs/command-line-interface.html#config).

## Plugin settings

For plugin settings, refer to the specific plugin's documentation
([extractors](https://hub.meltano.com/extractors/), [loaders](https://hub.meltano.com/loaders/)),
or use [`meltano config <plugin> list`](/docs/command-line-interface.html#config)
to list all available settings with their names, environment variables, and current values.

## Your Meltano project

These are settings specific to [your Meltano project](/docs/project.html).

### `send_anonymous_usage_stats`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_SEND_ANONYMOUS_USAGE_STATS`, alias: `!MELTANO_DISABLE_TRACKING` (implies value `false`)
- [`meltano init`](/docs/command-line-interface.html#init) CLI option: `--no_usage_stats` (implies value `false`)
- Default: `true`

By default, Meltano shares anonymous usage data with the Meltano team using Google Analytics. We use this data to learn about the size of our user base and the specific Meltano features they are (not yet) using, which helps us determine the highest impact changes we can make in each weekly release to make Meltano even more useful for you and others like you.

If enabled, Meltano will use the value of the [`project_id` setting](#project-id) to uniquely identify your project in Google Analytics.
This project ID is also sent along when Meltano loads the remote `discovery.yml` manifest from the URL identified by the [`discovery_url` setting](#discovery-url).

If you'd like to send the tracking data to a different Google Analytics account than the one run by the Meltano team,
the Tracking IDs can be configured using the [`tracking_ids.*` settings](#analytics-tracking-ids) below.

If you'd prefer to use Meltano _without_ sending the team this kind of data, you can disable tracking entirely using one of these methods:

- When creating a new project, pass `--no_usage_stats` to [`meltano init`](/docs/command-line-interface.html#init)
- In an existing project, disable this `send_anonymous_usage_stats` setting
- To disable tracking in all projects in one go, enable the `MELTANO_DISABLE_TRACKING` environment variable

When anonymous usage tracking is enabled, Meltano tracks the following events:

- `meltano init {project name}`
- `meltano ui`
- `meltano elt {extractor} {loader} --transform {skip, only, run}`
- `meltano add {extractor, loader, transform, model, transformer, orchestrator}`
- `meltano discover {all, extractors, loaders, transforms, models, transformers, orchestrators}`
- `meltano install`
- `meltano invoke {plugin_name} {plugin_args}`
- `meltano select {extractor} {entities_filter} {attributes_filter}`
- `meltano schedule add {name} {extractor} {loader} {interval}`

Beyond the invocation of these commands and the identified command line arguments, Meltano does not track any other event metadata, plugin configuration, or processed data.

Finally, Meltano also tracks anonymous web metrics when browsing the Meltano UI pages.

#### How to use

```bash
meltano config meltano set send_anonymous_usage_stats false

export MELTANO_SEND_ANONYMOUS_USAGE_STATS=false
export MELTANO_DISABLE_TRACKING=true

meltano init --no_usage_stats demo-project
```

### `project_id`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_PROJECT_ID`
- Default: None

Used by Meltano to uniquely identify your project in Google Analytics if the [`send_anonymous_usage_stats` setting](#send-anonymous-usage-stats) is enabled.

#### How to use

```bash
meltano config meltano set project_id <randomly-generated-token>

export MELTANO_PROJECT_ID=<randomly-generated-token>
```

### `database_uri`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_DATABASE_URI`
- `meltano *` CLI option: `--database-uri`
- Default: `sqlite:///$MELTANO_PROJECT_ROOT/.meltano/meltano.db`

Meltano stores various types of metadata in a project-specific [system database](/docs/project.html#system-database),
that takes the shape of a SQLite database stored inside the [`.meltano` directory](/docs/project.html#meltano-directory) at `.meltano/meltano.db` by default.

You can choose to use a different system database backend or configuration using the `--database-uri`
option of [`meltano` subcommands](/docs/command-line-interface.html), or the `MELTANO_DATABASE_URI` environment variable.

#### How to use

```bash
meltano config meltano set database_uri postgresql://<username>:<password>@<host>:<port>/<database>

export MELTANO_DATABASE_URI=postgresql://<username>:<password>@<host>:<port>/<database>

meltano elt --database-uri=postgresql://<username>:<password>@<host>:<port>/<database> ...
```

#### Targeting a PostgreSQL Schema

When using PostgreSQL as your [system database](/docs/project.html#system-database), you can choose the target schema within that database by adding
`?options=-csearch_path%3D<schema>` directly to the end of your `database_uri` and `MELTANO_DATABASE_URI`.

You are also able to add multiple schemas, which PostgreSQL will work through from left to right until it finds a valid schema to target, by using `?options=-csearch_path%3D<schema>,<schema_two>`

If you dont target a schema then by default PostgreSQL will try to use the `public` schema.

```bash
postgresql://<username>:<password>@<host>:<port>/<database>?options=-csearch_path%3D<schema>
```


### `database_max_retries`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_DATABASE_MAX_RETRIES`
- Default: `3`

This sets the maximum number of reconnection attempts in case the initial connection to the database fails because it isn't available when Meltano starts up.

Note: This affects the initial connection attempt only after which the connection is cached.
Subsequent disconnections are handled by SQLALchemy

#### How to use

```bash
meltano config meltano set database_max_retries 3

export MELTANO_DATABASE_MAX_RETRIES=3
```

### `database_retry_timeout`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_DATABASE_RETRY_TIMEOUT`
- Default: `5` (seconds)

This controls the retry interval (in seconds) in case the initial connection to the database fails because it isn't available when Meltano starts up.

Note: This affects the initial connection attempt only after which the connection is cached.
Subsequent disconnections are handled by SQLALchemy

#### How to use

```bash
meltano config meltano set database_retry_timeout 5

export MELTANO_DATABASE_RETRY_TIMEOUT=5
```

### `project_readonly`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_PROJECT_READONLY`
- Default: `false`

Enable this setting to indicate that your Meltano project is deployed as read-only,
and to block all modifications to project files through the [CLI](/docs/command-line-interface.md) and [UI](/docs/command-line-interface.md#ui)
in this environment.

Specifically, this prevents [adding plugins](/docs/command-line-interface.md#add) or [pipeline schedules](/docs/command-line-interface.md#schedule) to your [`meltano.yml` project file](/docs/project.html#meltano-yml-project-file), as well as [modifying plugin configuration](/docs/command-line-interface.md#config) stored in [`meltano.yml`](/docs/project.html#meltano-yml-project-file) or [`.env`](/docs/project.html#env).

Note that [`meltano config <plugin> set`](/docs/command-line-interface.md#config) and [the UI](/docs/ui.html)
can still be used to store configuration in the [system database](/docs/project.html#system-database),
but that settings that are already [set in the environment](/docs/configuration.html#configuring-settings) or `meltano.yml` take precedence and cannot be overridden.

This setting differs from the [`ui.readonly` setting](#ui-readonly) in two ways:
1. it does not block write actions in the UI that do not modify project files, like storing settings in the [system database](/docs/project.html#system-database), and
2. it also affects the [CLI](/docs/command-line-interface.md).

#### How to use

```bash
meltano config meltano set project_readonly true

export MELTANO_PROJECT_READONLY=true
```

### `discovery_url`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_DISCOVERY_URL`
- Default: [`https://www.meltano.com/discovery.yml`](https://www.meltano.com/discovery.yml)

Where Meltano can find the `discovery.yml` manifest that lists all [discoverable plugins](/docs/plugins.html#discoverable-plugins) that are supported out of the box.

This manifest is used by [`meltano discover`](/docs/command-line-interface.md#discover) and [`meltano add`](/docs/command-line-interface.md#add), among others.

To disable downloading the remote `discovery.yml` manifest and only use the project-local or packaged version,
set this setting to `false` or any other string not starting with `http://` or `https://`.

#### How to use

```bash
meltano config meltano set discovery_url https://meltano.example.com/discovery.yml
meltano config meltano set discovery_url false

export MELTANO_DISCOVERY_URL=https://meltano.example.com/discovery.yml
export MELTANO_DISCOVERY_URL=false
```

### `discovery_url_auth`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_DISCOVERY_URL_AUTH`
- Default: None

The value of the `Authorization` header sent when making a request to [`discovery_url`](#discovery-url).

No `Authorization` header is applied under the following conditions:
- `discovery_url_auth` is not set
- `discovery_url_auth` is set to `false`, `null` or an empty string

#### How to use

```bash
meltano config meltano set discovery_url_auth "Bearer $ACCESS_TOKEN"
meltano config meltano set discovery_url_auth false

export MELTANO_DISCOVERY_URL_AUTH="Bearer $ACCESS_TOKEN"
export MELTANO_DISCOVERY_URL_AUTH=false
```

## `meltano` CLI

These settings can be used to modify the behavior of the [`meltano` CLI](/docs/command-line-interface.html).

### `cli.log_level`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_CLI_LOG_LEVEL`, alias: `MELTANO_LOG_LEVEL`
- `meltano` CLI option: `--log-level`
- Options: `debug`, `info`, `warning`, `error`, `critical`
- Default: `info`

The granularity of CLI logging.

#### How to use

```bash
meltano config meltano set cli log_level debug

export MELTANO_CLI_LOG_LEVEL=debug
export MELTANO_LOG_LEVEL=debug

meltano --log-level=debug ...
```

## `meltano elt`

These settings can be used to modify the behavior of [`meltano elt`](/docs/command-line-interface.html#elt).

### `elt.buffer_size`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_ELT_BUFFER_SIZE`
- Default: `10485760` (10MiB in bytes)

Size (in bytes) of the buffer between extractor and loader (Singer tap and target) that stores
[messages](https://hub.meltano.com/singer/spec#messages)
output by the extractor while they are waiting to be processed by the loader.

When an extractor generates messages (records) faster than the loader can process them, the buffer may fill up completely,
at which point the extractor will be blocked until the loader has worked through enough messages to make half
of the buffer size available again for new extractor output.

The length of a single line of extractor output is limited to half the buffer size.
With a default buffer size of 10MiB, the maximum message size would therefore be 5MiB.

#### How to use

```bash
meltano config meltano set elt.buffer_size 52428800 # 50MiB in bytes

export MELTANO_ELT_BUFFER_SIZE=52428800
```

## Meltano UI server

These settings can be used to configure the [Meltano UI](/docs/ui.html) server.

[Meltano UI feature settings](#meltano-ui-features) and [customization settings](#meltano-ui-customization) have their own sections.

### `ui.bind_host`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_BIND_HOST`, alias: `MELTANO_API_HOSTNAME`
- [`meltano ui`](/docs/command-line-interface.html#ui) CLI option: `--bind`
- Default: `0.0.0.0`

The host to bind to.

Together with the [`ui.bind_port` setting](#ui-bind-port), this setting corresponds to
[Gunicorn's `bind` setting](https://docs.gunicorn.org/en/stable/settings.html#bind).

#### How to use

```bash
meltano config meltano set ui bind_host 127.0.0.1

export MELTANO_UI_BIND_HOST=127.0.0.1
export MELTANO_API_HOSTNAME=127.0.0.1

meltano ui --bind=127.0.0.1
```

### `ui.bind_port`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_BIND_PORT`, alias: `MELTANO_API_PORT`, `PORT`
- [`meltano ui`](/docs/command-line-interface.html#ui) CLI option: `--bind-port`
- Default: `5000`

The port to bind to.

Together with the [`ui.bind_host` setting](#ui-bind-host), this setting corresponds to
[Gunicorn's `bind` setting](https://docs.gunicorn.org/en/stable/settings.html#bind).

#### How to use

```bash
meltano config meltano set ui bind_port 80

export MELTANO_UI_BIND_PORT=80
export MELTANO_API_PORT=80
export PORT=80

meltano ui --bind-port=80
```

### `ui.server_name`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_SERVER_NAME`
- Default: None

The host and port Meltano UI is available at, e.g. `<host>:<port>`.

The port will usually match the [`ui.bind_port` setting](#ui-bind-port), and can be omitted when the default port for HTTP (`80`) or HTTPS (`443`) is used.

Unless the [`ui.session_cookie_domain` setting](#ui-session-cookie-domain) is set, this setting will be used as the session cookie domain.

If the [`ui.notification` setting](#ui-notification) is enabled, this setting will be used to generate external URLs in notification emails.

When set, Meltano UI will only respond to requests whose hostname (`Host` header) matches this setting.
If this is undesirable, you can set the [`ui.session_cookie_domain` setting](#ui-session-cookie-domain) instead.
This may be the case when Meltano UI is situated behind a load balancer performing health checks without specifying a hostname.

If the [`ui.authentication` setting](#ui-authentication) is enabled,
[`meltano ui`](/docs/command-line-interface.html#ui) will print a
security warning if neither this setting or the [`ui.session_cookie_domain` setting](#ui-session-cookie-domain) has been set.

This setting corresponds to [Flask's `SERVER_NAME` setting](https://flask.palletsprojects.com/en/1.1.x/config/#SERVER_NAME).

#### How to use

```bash
meltano config meltano set ui server_name meltano.example.com

export MELTANO_UI_SERVER_NAME=meltano.example.com
```

[`meltano ui setup <server_name>`](/docs/command-line-interface.html#setup) can be
used to generate secrets for the [`ui.secret_key`](#ui-secret-key) and
[`ui.password_salt`](#ui-password-salt) settings, that will be stored in a
your project's [`.env` file](/docs/project.html#env) along with the specified `server_name`.

```bash
meltano ui setup meltano.example.com
```

### `ui.session_cookie_domain`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_SESSION_COOKIE_DOMAIN`
- Default: None

The domain match rule that the session cookie will be valid for.

If not set, the cookie will be valid for all subdomains of the configured [`ui.server_name`](#ui-server-name).

If the [`ui.authentication` setting](#ui-authentication) is enabled,
[`meltano ui`](/docs/command-line-interface.html#ui) will print a
security warning if neither this setting or the [`ui.server_name` setting](#ui-server-name) has been set.

This setting corresponds to [Flask's `SESSION_COOKIE_DOMAIN` setting](https://flask.palletsprojects.com/en/1.1.x/config/#SESSION_COOKIE_DOMAIN).

#### How to use

```bash
meltano config meltano set ui session_cookie_domain meltano.example.com

export MELTANO_UI_SESSION_COOKIE_DOMAIN=meltano.example.com
```

### `ui.session_cookie_secure`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_SESSION_COOKIE_SECURE`
- Default: `false`

Enable the `Secure` flag on the session cookie, so that the client will only send it to the server in HTTPS requests.

The application must be served over HTTPS for this to make sense.

This setting corresponds to [Flask's `SESSION_COOKIE_SECURE` setting](https://flask.palletsprojects.com/en/1.1.x/config/#SESSION_COOKIE_SECURE).

#### How to use

```bash
meltano config meltano set ui session_cookie_secure true

export MELTANO_UI_SESSION_COOKIE_SECURE=true
```

### `ui.secret_key`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_SECRET_KEY`
- Default: `thisisnotapropersecretkey`

A secret key that will be used for securely signing the session cookie.

If the [`ui.authentication` setting](#ui-authentication) is enabled,
[`meltano ui`](/docs/command-line-interface.html#ui) will print a
security warning if this setting has not been changed from the default.

This setting corresponds to [Flask's `SECRET_KEY` setting](https://flask.palletsprojects.com/en/1.1.x/config/#SECRET_KEY).

#### How to use

```bash
meltano config meltano set ui secret_key <randomly-generated-secret>

export MELTANO_UI_SECRET_KEY=<randomly-generated-secret>
```

[`meltano ui setup <server_name>`](/docs/command-line-interface.html#setup) can be
used to generate secrets for the this setting and [`ui.password_salt`](#ui-password-salt),
that will be stored in your project's [`.env` file](/docs/project.html#env)
along with the specified [`ui.server_name`](#ui-server-name).

```bash
meltano ui setup meltano.example.com
```

### `ui.password_salt`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_PASSWORD_SALT`
- Default: `b4c124932584ad6e69f2774a0ae5c138`

The HMAC salt to use when hashing passwords.

If the [`ui.authentication` setting](#ui-authentication) is enabled,
[`meltano ui`](/docs/command-line-interface.html#ui) will print a
security warning if this setting has not been changed from the default.

This setting corresponds to [Flask-Security's `SECURITY_PASSWORD_SALT` setting](https://pythonhosted.org/Flask-Security/configuration.html).

#### How to use

```bash
meltano config meltano set ui password_salt <randomly-generated-secret>

export MELTANO_UI_PASSWORD_SALT=<randomly-generated-secret>
```

[`meltano ui setup <server_name>`](/docs/command-line-interface.html#setup) can be
used to generate secrets for the this setting and [`ui.secret_key`](#ui-secret-key),
that will be stored in your project's [`.env` file](/docs/project.html#env)
along with the specified [`ui.server_name`](#ui-server-name).

```bash
meltano ui setup meltano.example.com
```

### `ui.workers`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_WORKERS`, alias: `WORKERS`, `WEB_CONCURRENCY`
- Default: `4`

The number of worker processes `meltano ui` will use to handle requests.

This setting corresponds to [Gunicorn's `workers` setting](https://docs.gunicorn.org/en/stable/settings.html#workers).

#### How to use

```bash
meltano config meltano set ui workers 1

export MELTANO_UI_WORKERS=1
export WORKERS=1
export WEB_CONCURRENCY=1
```

### `ui.forwarded_allow_ips`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_FORWARDED_ALLOW_IPS`, alias: `FORWARDED_ALLOW_IPS`
- Default: `127.0.0.1`

Comma-separated front-end (reverse) proxy IPs that are allowed to set secure headers to indicate HTTPS requests.

Set to `*` to disable checking of front-end IPs, which can be useful for setups where you don't know in advance the IP address of front-end, but you still trust the environment.

This setting corresponds to [Gunicorn's `forwarded_allow_ips` setting](https://docs.gunicorn.org/en/stable/settings.html#forwarded-allow-ips).

#### How to use

```bash
meltano config meltano set ui forwarded_allow_ips "*"

export MELTANO_UI_FORWARDED_ALLOW_IPS="*"
export FORWARDED_ALLOW_IPS="*"
```

## Meltano UI features

These settings can be used to enable certain features of [Meltano UI](/docs/ui.html).

[Meltano UI server settings](#meltano-ui-server) and [customization settings](#meltano-ui-customization) have their own sectionss

### `ui.readonly`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_READONLY`, alias: `MELTANO_READONLY`
- Default: `false`

To block all write actions in the Meltano UI, you can run it in in *read-only* mode.

If you're enabling the [`ui.authentication` setting](#ui-authentication) and would
like to only use read-only mode for anonymous users, enable the [`ui.anonymous_readonly` setting](#ui-anonymous-readonly) instead.

This setting differs from the [`project_readonly` setting](#project-readonly) in two ways:
1. it also blocks write actions in the UI that do not modify project files, like storing settings in the [system database](/docs/project.html#system-database), and
2. it does not affect the [CLI](/docs/command-line-interface.md).

#### How to use

```bash
meltano config meltano set ui readonly true

export MELTANO_UI_READONLY=true
export MELTANO_READONLY=true
```

### `ui.authentication`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_AUTHENTICATION`, alias: `MELTANO_AUTHENTICATION`
- Default: `false`

Use this setting to enable authentication and disallow anonymous usage of your Meltano instance.

Additionally, you will need to:
1. Ensure your configuration is secure by setting the [`ui.secret_key`](#ui-secret-key) and [`ui.password_salt`](#ui-password-salt) settings, as well as [`ui.server_name`](#ui-server-name) or [`ui.session_cookie_domain`](#ui-session-cookie-domain), manually or using [`meltano ui setup <server_name>`](./command-line-interface.html#setup).

2. Create at least one user using [`meltano user add`](./command-line-interface.html#user).

#### How to use

```bash
meltano config meltano set ui authentication true

export MELTANO_UI_AUTHENTICATION=true
export MELTANO_AUTHENTICATION=true
```

### `ui.anonymous_readonly`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_ANONYMOUS_READONLY`
- Default: `false`

When the [`ui.authentication` setting](#ui-authentication) is enabled,
enabling this setting will allow anonymous users read-only access to Meltano UI.
Once a user is authenticated, write actions will be available again.

This setting is especially useful when setting up a publicly available demo
instance of Meltano UI for anonymous users to interact with.
These users will not be able to make any changes, but admins will once they sign in.

#### How to use

```bash
meltano config meltano set ui anonymous_readonly true

export MELTANO_UI_ANONYMOUS_READONLY=true
```

### `ui.notification`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_NOTIFICATION`, alias: `MELTANO_NOTIFICATION`
- Default: `false`

Meltano can send email notifications upon certain events.

Your outgoing mail server can be configured using the [`mail.*` settings](#mail-server) below.

::: tip
To ease the development and testing, Meltano is preconfigured to use a local [MailHog](https://github.com/mailhog) instance to trap all the outgoing emails.

Use the following docker command to start it:

```bash
docker run --rm -p 1025:1025 -p 8025:8025 --name mailhog mailhog/mailhog
```

All emails sent by Meltano should now be available at `http://localhost:8025/`
:::

#### How to use

```bash
meltano config meltano set ui notification true

export MELTANO_UI_NOTIFICATION=true
export MELTANO_NOTIFICATION=true
```

### `ui.analysis`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_ANALYSIS`
- Default: `true`

If you are only using Meltano for data integration (and transformation),
you can disable this setting to hide all functionality related to Analysis from the UI:
- "Explore" and "Dashboards" tabs
- "Explore" buttons in the "Pipelines" list and "Pipeline Run Log" modal

#### How to use

```bash
meltano config meltano set ui analysis false

export MELTANO_UI_ANALYSIS=true
```

## Meltano UI customization

These settings can be used to customize certain aspects of [Meltano UI](/docs/ui.html).

[Meltano UI server settings](#meltano-ui-server) and [feature settings](#meltano-ui-features) have their own sections.

### `ui.logo_url`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_UI_LOGO_URL`
- Default: None

Customize the logo used by Meltano UI in the navigation bar and on the sign-in page (when the [`ui.authentication` setting](#ui-authentication) is enabled).

#### How to use

```bash
meltano config meltano set ui logo_url https://meltano.com/meltano-logo-with-text.svg

export MELTANO_UI_LOGO_URL=https://meltano.com/meltano-logo-with-text.svg
```

## Mail server

Meltano uses [Flask-Mail](https://pythonhosted.org/Flask-Mail/) to send emails. Take a look at the documentation to properly configure your outgoing email server.

### `mail.server`

- [Environment variable](/docs/configuration.html#configuring-settings): `MAIL_SERVER`
- Default: `localhost`

```bash
meltano config meltano set mail server smtp.example.com

export MAIL_SERVER=smtp.example.com
```

### `mail.port`

- [Environment variable](/docs/configuration.html#configuring-settings): `MAIL_PORT`
- Default: `1025`

```bash
meltano config meltano set mail port 25

export MAIL_PORT=25
```

### `mail.default_sender`

- [Environment variable](/docs/configuration.html#configuring-settings): `MAIL_DEFAULT_SENDER`
- Default: `"Meltano" <bot@meltano.com>`

```bash
meltano config meltano set mail default_sender '"Example Meltano" <bot@meltano.example.com>'

export MAIL_DEFAULT_SENDER='"Example Meltano" <bot@meltano.example.com>'
```

### `mail.use_tls`

- [Environment variable](/docs/configuration.html#configuring-settings): `MAIL_USE_TLS`
- Default: `false`

```bash
meltano config meltano set mail use_tls true

export MAIL_USE_TLS=true
```

### `mail.username`

- [Environment variable](/docs/configuration.html#configuring-settings): `MAIL_USERNAME`
- Default: None

```bash
meltano config meltano set mail username meltano

export MAIL_USERNAME=meltano
```

### `mail.password`

- [Environment variable](/docs/configuration.html#configuring-settings): `MAIL_PASSWORD`
- Default: None

```bash
meltano config meltano set mail password meltano

export MAIL_PASSWORD=meltano
```

### `mail.debug`

- [Environment variable](/docs/configuration.html#configuring-settings): `MAIL_DEBUG`
- Default: `false`

```bash
meltano config meltano set mail debug true

export MAIL_DEBUG=true
```

## OAuth Service

Meltano ships with an OAuth Service to handle the OAuth flow in the Extractors' configuration.

::: warning
To run this service, you **must** have a registered OAuth application on the [Authorization server](https://www.oauth.com/oauth2-servers/definitions/#the-authorization-server).

Most importantly, the Redirect URI must be set properly so that the OAuth flow can be completed.

This process is specific to each Provider.
:::

The OAuth Service is bundled within Meltano, and is automatically started with [`meltano ui`](/docs/command-line-interface.html#ui) and mounted at `/-/oauth` for development purposes.

As it is a Flask application, it can also be run as a standalone using:

```bash
FLASK_ENV=production FLASK_APP=meltano.oauth python -m flask run --port 9999
```

### `oauth_service.url`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_OAUTH_SERVICE_URL`
- Default: None

Meltano provides a public hosted solution at <https://oauth.svc.meltanodata.com>.

The local OAuth service for development purposes is available at `/-/oauth`.

#### How to use

```bash
meltano config meltano set oauth_service url https://oauth.svc.meltanodata.com

export MELTANO_OAUTH_SERVICE_URL=https://oauth.svc.meltanodata.com
```

### `oauth_service.providers`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_OAUTH_SERVICE_PROVIDERS`
- Default: `all`

To enable specific providers, use comma-separated `oauth.provider` names from `discovery.yml`. To enable all providers, use `all`.

#### How to use

```bash
meltano config meltano set oauth_service providers facebook,google_adwords

export MELTANO_OAUTH_SERVICE_PROVIDERS=facebook,google_adwords
```

### `oauth_service.facebook.client_id`

- [Environment variable](/docs/configuration.html#configuring-settings): `OAUTH_FACEBOOK_CLIENT_ID`
- Default: None

```bash
meltano config meltano set oauth_service facebook client_id <facebook-client-id>

export OAUTH_FACEBOOK_CLIENT_ID=<facebook-client-id>
```

### `oauth_service.facebook.client_secret`

- [Environment variable](/docs/configuration.html#configuring-settings): `OAUTH_FACEBOOK_CLIENT_SECRET`
- Default: None

```bash
meltano config meltano set oauth_service facebook client_secret <facebook-client-secret>

export OAUTH_FACEBOOK_CLIENT_SECRET=<facebook-client-secret>
```

### `oauth_service.google_adwords.client_id`

- [Environment variable](/docs/configuration.html#configuring-settings): `OAUTH_GOOGLE_ADWORDS_CLIENT_ID`
- Default: None

```bash
meltano config meltano set oauth_service google_adwords client_id <google-adwords-client-id>

export OAUTH_GOOGLE_ADWORDS_CLIENT_ID=<google-adwords-client-id>
```

### `oauth_service.google_adwords.client_secret`

- [Environment variable](/docs/configuration.html#configuring-settings): `OAUTH_GOOGLE_ADWORDS_CLIENT_SECRET`
- Default: None

```bash
meltano config meltano set oauth_service google_adwords client_secret <google-adwords-client-secret>

export OAUTH_GOOGLE_ADWORDS_CLIENT_SECRET=<google-adwords-client-secret>
```

## OAuth Single-Sign-On

These variables are specific to [Flask-OAuthlib](https://flask-oauthlib.readthedocs.io/en/latest/#) and work with [OAuth authentication with GitLab](https://docs.gitlab.com/ee/integration/oauth_provider.html).

::: tip
These settings are used for single-sign-on using an external OAuth provider.
:::

For more information on how to get these from your GitLab application, check out the [integration docs from GitLab](https://docs.gitlab.com/ee/integration/gitlab.html).

### `oauth.gitlab.client_id`

- [Environment variable](/docs/configuration.html#configuring-settings): `OAUTH_GITLAB_CLIENT_ID`, alias: `OAUTH_GITLAB_APPLICATION_ID`
- Default: None

```bash
meltano config meltano set oauth gitlab client_id <gitlab-client-id>

export OAUTH_GITLAB_CLIENT_ID=<gitlab-client-id>
export OAUTH_GITLAB_APPLICATION_ID=<gitlab-client-id>
```

### `oauth.gitlab.client_secret`

- [Environment variable](/docs/configuration.html#configuring-settings): `OAUTH_GITLAB_CLIENT_SECRET`, alias: `OAUTH_GITLAB_SECRET`
- Default: None

```bash
meltano config meltano set oauth gitlab client_secret <gitlab-client-secret>

export OAUTH_GITLAB_CLIENT_SECRET=<gitlab-client-secret>
export OAUTH_GITLAB_SECRET=<gitlab-client-secret>
```

## Analytics Tracking IDs

Google Analytics Tracking IDs to be used if the [`send_anonymous_usage_stats` setting](#send-anonymous-usage-stats) is enabled.

### `tracking_ids.cli`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_TRACKING_IDS_CLI`, alias: `MELTANO_CLI_TRACKING_ID`
- Default: `UA-132758957-3`

Tracking ID for usage of the [`meltano` CLI](/docs/command-line-interface.html).

```bash
meltano config meltano set tracking_ids cli UA-123456789-1

export MELTANO_TRACKING_IDS_CLI=UA-123456789-1
export MELTANO_CLI_TRACKING_ID=UA-123456789-1
```

### `tracking_ids.ui`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_TRACKING_IDS_UI`, alias: `MELTANO_UI_TRACKING_ID`
- Default: `UA-132758957-2`

Tracking ID for usage of [Meltano UI](/docs/ui.html).

```bash
meltano config meltano set tracking_ids ui UA-123456789-2

export MELTANO_TRACKING_IDS_UI=UA-123456789-2
export MELTANO_UI_TRACKING_ID=UA-123456789-2
```

### `tracking_ids.ui_embed`

- [Environment variable](/docs/configuration.html#configuring-settings): `MELTANO_TRACKING_IDS_UI_EMBED`, alias: `MELTANO_EMBED_TRACKING_ID`
- Default: `UA-132758957-6`

Tracking ID for usage of [Meltano UI](/docs/ui.html)'s [Embed feature](/docs/analysis.html#share-reports-and-dashboards).

```bash
meltano config meltano set tracking_ids ui_embed UA-123456789-3

export MELTANO_TRACKING_IDS_UI_EMBED=UA-123456789-3
export MELTANO_EMBED_TRACKING_ID=UA-123456789-3
```
