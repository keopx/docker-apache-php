# Introduction

On current version we used new template system to setup configuration properly. 

# Customizable values

We can see different properties with default values.

## Environment values for docker-compose.yml

To work correctly variables you need to update `docker-compose.yml` file with environment variables:

Change on `.env` file:

```ini
XDEBUG_ENABLED=1
```

And create environment variable on  `docker-compose.yml`:

```yml
    environment:
      ## Use webroot directory. i.e: /web for drupal scaffolding.
      - WEB_ROOT
      ## SSH support. Uncomment volumes -> # - ${SSH_AUTH_SOCK}:/ssh-agent
      - SSH_AUTH_SOCK=/ssh-agent
      ### WARNING: Use only if you not use custom php.ini.
      - PHP_SENDMAIL_PATH
      - PHP_SENDMAIL_DOMAIN
      - XDEBUG_ENABLED
```

## Environment configuration

The default configuration is __not recommended to be used for production__ environment:

### PHP configuration

Values are based on `php.ini-development` file.

| Variable                        | 7.3                  | 7.2                  | 7.1                  | 7.0                  | 5.6                  |
| ------------------------------- | -------------------- | -------------------- | -------------------- | -------------------- | -------------------- |
| [`PHP_DATE_TIMEZONE`]           | `Europe/Berlin`      | `Europe/Berlin`      | `Europe/Berlin`      | `Europe/Berlin`      | `Europe/Berlin`      |
| [`PHP_DISPLAY_ERRORS`]          | `On`                 | `On`                 | `On`                 | `On`                 | `On`                 |
| [`PHP_DISPLAY_STARTUP_ERRORS`]  | `On`                 | `On`                 | `On`                 | `On`                 | `On`                 |
| [`PHP_ERROR_REPORTING`]         | `E_ALL`              | `E_ALL`              | `E_ALL`              | `E_ALL`              | `E_ALL`              |
| [`PHP_EXPOSE`]                  | `On`                 | `On`                 | `On`                 | `On`                 | `On`                 |
| [`PHP_LOG_ERRORS`]              | `On`                 | `On`                 | `On`                 | `On`                 | `On`                 |
| [`PHP_LOG_ERRORS_MAX_LEN`]      | `1024`               | `1024`               | `1024`               | `1024`               | `1024`               |
| [`PHP_MAX_EXECUTION_TIME`]      | `30`                 | `30`                 | `30`                 | `30`                 | `30`                 |
| [`PHP_MAX_FILE_UPLOADS`]        | `20`                 | `20`                 | `20`                 | `20`                 | `20`                 |
| [`PHP_MAX_INPUT_TIME`]          | `60`                 | `60`                 | `60`                 | `60`                 | `60`                 |
| [`PHP_MAX_INPUT_VARS`]          | `2000`               | `2000`               | `2000`               | `2000`               | `2000`               |
| [`PHP_MEMORY_LIMIT`]            | `128M`               | `128M`               | `128M`               | `128M`               | `128M`               |
| [`PHP_POST_MAX_SIZE`]           | `8M`                 | `8M`                 | `8M`                 | `8M`                 | `8M`                 |
| [`PHP_SENDMAIL_PATH`]           | `/usr/sbin/ssmtp -t` | `/usr/sbin/ssmtp -t` | `/usr/sbin/ssmtp -t` | `/usr/sbin/ssmtp -t` | `/usr/sbin/ssmtp -t` |
| [`PHP_TRACK_ERRORS`]            | -                    | -                    | `On`                 | `On`                 | `On`                 |
| [`PHP_UPLOAD_MAX_FILESIZE`]     | `2M`                 | `2M`                 | `2M`                 | `2M`                 | `2M`                 |
| [`PHP_ZEND_ASSERTIONS`]         | `1`                  | `1`                  | `1`                  | `1`                  | `1`                  |
| [`XDEBUG_HOST`]                 | `localhost`          | `localhost`          | `localhost`          | `localhost`          | `localhost`          |

### PHP XDEBUG configuration

| Variable                           | Conf        |
| ---------------------------------- | ----------- |
| [`XDEBUG_ENABLED`]                 | `1`         |
| [`XDEBUG_REMOTE_CONNECT_BACK`]     | `1`         |
| [`XDEBUG_REMOTE_AUTOSTART`]        | `1`         |
| [`XDEBUG_REMOTE_ENABLE`]           | `1`         |
| [`XDEBUG_REMOTE_PORT`]             | `9000`      |
| [`XDEBUG_MAX_NESTING_LEVEL`]       | `500`       |
| [`XDEBUG_IDEKEY`]                  | `PHPSTORM`  |
| [`XDEBUG_PROFILER_ENABLE_TRIGGER`] | `1`         |
| [`XDEBUG_SHOW_ERROR_TRACE`]        | `1`         |

### sSMTP configuration

| Variable                 | Conf        |
| ------------------------ | ----------- |
| [`PHP_SENDMAIL_DOMAIN`]  | `true`      |

### Vhost configuration

| Variable                | Conf            |
| ----------------------- | --------------- |
| [`DEFAULT_ROOT`]        | `/var/www/html` |
| [`WEB_ROOT`]            | ` `             |
| [`APACHE_SERVER_NAME`]  | `localhost`     |

## Template customizable values

We can see different properties with default values.

### php.ini

Values are based on `php.ini-development` file.

```ini
[PHP]
expose_php = {{ getenv "PHP_EXPOSE" "On" }}
error_reporting = {{ getenv "PHP_ERROR_REPORTING" "E_ALL" }}
display_errors = {{ getenv "PHP_DISPLAY_ERRORS" "On" }}
display_startup_errors = {{ getenv "PHP_DISPLAY_STARTUP_ERRORS" "On" }}
log_errors = {{ getenv "PHP_LOG_ERRORS" "On" }}
log_errors_max_len = {{ getenv "PHP_LOG_ERRORS_MAX_LEN" "1024" }}

max_execution_time = {{ getenv "PHP_MAX_EXECUTION_TIME" "30" }}
max_input_time = {{ getenv "PHP_MAX_INPUT_TIME" "60" }}
max_input_vars = {{ getenv "PHP_MAX_INPUT_VARS" "2000" }}
post_max_size = {{ getenv "PHP_POST_MAX_SIZE" "8M" }}
upload_max_filesize = {{ getenv "PHP_UPLOAD_MAX_FILESIZE" "2M" }}
max_file_uploads = {{ getenv "PHP_MAX_FILE_UPLOADS" "20" }}
memory_limit = {{ getenv "PHP_MEMORY_LIMIT" "128M" }}

[Date]
date.timezone = {{ getenv "PHP_DATE_TIMEZONE" "Europe/Berlin"}}

[Assertion]
zend.assertions = {{ getenv "PHP_ZEND_ASSERTIONS" "1" }}

[mail function]
sendmail_path = {{ getenv "PHP_SENDMAIL_PATH" "/usr/sbin/ssmtp -t" }}

[Xdebug]
xdebug.remote_host = {{ getenv "XDEBUG_HOST" "localhost" }}
```

### xdebug.ini

```ini
xdebug.default_enable = {{ getenv "XDEBUG_ENABLED" "1" }}
xdebug.remote_connect_back = {{ getenv "XDEBUG_REMOTE_CONNECT_BACK" "1" }}
xdebug.remote_autostart = {{ getenv "XDEBUG_REMOTE_AUTOSTART" "1" }}
xdebug.remote_enable = {{ getenv "XDEBUG_REMOTE_ENABLE" "1" }}
xdebug.remote_host = {{ getenv "XDEBUG_HOST" "localhost" }}
xdebug.remote_port = {{ getenv "XDEBUG_REMOTE_PORT" "9000" }}
xdebug.max_nesting_level = {{ getenv "XDEBUG_MAX_NESTING_LEVEL" "500" }}
xdebug.idekey = {{ getenv "XDEBUG_IDEKEY" "PHPSTORM" }}
xdebug.profiler_enable_trigger = {{ getenv "XDEBUG_PROFILER_ENABLE_TRIGGER" "1" }}
xdebug.show_error_trace = {{ getenv "XDEBUG_SHOW_ERROR_TRACE" "1" }}
```

### sSMTP

```ini
mailhub={{ getenv "PHP_SENDMAIL_DOMAIN" "true" }}
```

### Apache vhost

```ini
DocumentRoot {{ getenv "DEFAULT_ROOT" "/var/www/html" }}{{ getenv "WEB_ROOT" "" }}
ServerName {{ getenv "APACHE_SERVER_NAME" "localhost" }}
```
