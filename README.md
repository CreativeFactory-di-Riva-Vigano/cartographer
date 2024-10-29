# Cartographer

A sitemap generation tool for PHP following the [Sitemap Protocol v0.9](http://www.sitemaps.org/protocol.html), based on the work of Dan Horrigan ([tackk/cartographer](https://github.com/tackk/cartographer)). This fork was created to address vulnerabilities arising from outdated dependencies.

Cartographer can handle Sitemaps of any size.  When generating sitemaps with more than 50,000
entries, the sitemap becomes a "map of maps" (i.e. nested sitemaps).

## Supported PHP/HHVM Versions

* **PHP:** >= 8.0.2

## Installation

### Composer CLI

```
composer require creativefactoryrv/cartographer:1.0.*
```

### composer.json

``` json
{
    "require": {
        "creativefactoryrv/cartographer": "1.0.*"
    }
}
```

## Basic Sitemap

If you have a sitemap that is under 50,000 items, you can just use the Sitemap class, and avoid the Sitemap
Generator.

``` php
use CreativeFactoryRV\Cartographer\ChangeFrequency;
use CreativeFactoryRV\Cartographer\Sitemap

$sitemap = new Tackk\Cartographer\Sitemap();
$sitemap->add('http://foo.com', '2005-01-02', ChangeFrequency::WEEKLY, 1.0);
$sitemap->add('http://foo.com/about', '2005-01-01');

// Write it to a file
file_put_contents('sitemap.xml', (string) $sitemap);

// or simply echo it:
header ('Content-Type:text/xml');
echo $sitemap->toString();
```

### Output

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>http://foo.com</loc>
    <lastmod>2005-01-02T00:00:00+00:00</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1</priority>
  </url>
  <url>
    <loc>http://foo.com/about</loc>
    <lastmod>2005-01-01T00:00:00+00:00</lastmod>
  </url>
</urlset>
```

## Basic Sitemap Index

If you want to build a Sitemap Index, separate from the Sitemap Generator, you can!

``` php
$sitemapIndex = new CreativeFactoryRV\Cartographer\SitemapIndex();
$sitemapIndex->add('http://foo.com/sitemaps/sitemap.1.xml', '2012-01-02');
$sitemapIndex->add('http://foo.com/sitemaps/sitemap.2.xml', '2012-01-02');

// Write it to a file
file_put_contents('sitemap.xml', (string) $sitemapIndex);

// or simply echo it:
header ('Content-Type:text/xml');
echo $sitemapIndex->toString();
```

### Output

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>http://foo.com/sitemaps/sitemap.1.xml</loc>
    <lastmod>2012-01-02T00:00:00+00:00</lastmod>
  </url>
  <url>
    <loc>http://foo.com/sitemaps/sitemap.2.xml</loc>
    <lastmod>2012-01-02T00:00:00+00:00</lastmod>
  </url>
</sitemapindex>
```

## Sitemap Factory

The Sitemap Factory create Sitemaps and Sitemap Indexes and writes them to the Filesystem.
It can be used to generate full Sitemaps with more than **50,000** URLs.

If more than one sitemap is generated, it will create a Sitemap Index automatically.

### Instantiating

The factory uses [Flysystem](http://flysystem.thephpleague.com/) to write the sitemaps.  This
means you can write the sitemaps to Local Disk, S3, Dropbox, wherever you want.

``` php
<?php

use CreativeFactoryRV\Cartographer\SitemapFactory;
use League\Flysystem\Filesystem;
use League\Flysystem\Local\LocalFilesystemAdapter;

$adapter = new LocalFilesystemAdapter(__DIR__ . '/sitemaps');
$filesystem = new Filesystem($adapter);

$sitemapFactory = new SitemapFactory($filesystem);
```

### Base URL

The Base URL is used when generating the Sitemap Indexes, and for the returned entry point URL.

You can set the Base URL:

``` php
$sitemapFactory->setBaseUrl('http://foo.com/sitemaps/');
```

You can get the current base URL using `getBaseUrl()`.

### Creating a Sitemap

To create a sitemap you use the `createSitemap` method.  This method requires an `Iterator` as
its only parameter.

``` php
<?php

use CreativeFactoryRV\Cartographer\SitemapFactory;
use League\Flysystem\Filesystem;
use League\Flysystem\Local\LocalFilesystemAdapter;

$adapter = new LocalFilesystemAdapter(__DIR__ . '/sitemaps');
$filesystem = new Filesystem($adapter);

$sitemapFactory = new SitemapFactory($filesystem);
$sitemapFactory->setBaseUrl('http://foo.com/sitemaps/');

$entries = [
    [
        'url' => 'http://example.com/page1',
        'lastmod' => '2023-04-01',
        'changefreq' => 'daily',
        'priority' => '1.0'
    ],
    [
        'url' => 'http://example.com/page2',
        'lastmod' => '2023-04-02',
        'changefreq' => 'weekly',
        'priority' => '0.8'
    ],
];

$iterator = new ArrayIterator($entries);

// Returns the URL to the main Sitemap/Index file
$mainSitemap = $sitemapFactory->createSitemap($iterator);
```

### Return Value

The two creation methods (`createSitemap` and `createSitemapIndex`) will return the URL
of the root sitemap file.  If there is only 1 sitemap created, it will return just that URL.
If multiple sitemaps are created, then a Sitemap Index is generated and the URL to that is returned.

### List of Created Files

You can get a list (array) of files the Factory has created by using the `getFilesCreated` method.

``` php
$files = $sitemapFactory->getFilesCreated();
```

## Running Tests

*This assumes you ran `composer update`.*

From the repository root, run:

```
vendor/bin/phpunit
```
