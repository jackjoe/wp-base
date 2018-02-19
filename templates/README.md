# {{PROJECT_NAME}}

## Development

### Apache vhost

```
<VirtualHost 0.0.0.0:80>
  ServerName {{PROJECT_URL}}
  DocumentRoot "/path/to/{{PROJECT_SLUG}}"
</VirtualHost>
```

### Database

Create a local UTF8 database `{{PROJECT_SLUG}}-dev`

### Composer

#### Install bundle

```
$ composer install
```

#### Update bundle

If you want install additional plugins or update the WP core, update the composer.json file and run:

```
$ composer update
```

#### Add plugins

Search on [https://wpackagist.org/](https://wpackagist.org/) and add to composer.json and run the composer command again.
