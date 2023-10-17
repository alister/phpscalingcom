---
title: 'BunnyCDN via FlySystem on Symfony'
date: Tue, 17 Oct 2023 20:35:48 +0100
draft: false
tags: ['BunnyCDN','FlySystem','Symfony']
aliases: ['/2023/10/18/bunnycdn-via-flysystem-on-symfony/']
---

I've been refactoring the code at work to help move uploaded files off the main server, and also optimise the images when they are used <small>(no need to use a 4000 x 3000px 4.7MB file for a 500px wide thumbnail!)</small>. The standard library to upload files to a remote filesystem with PHP is [FlySystem](https://flysystem.thephpleague.com/docs/), it's been been easy to work with - even if it's not (yet) using Symfony on this project.

We are using [BunnyCDN](https://bunny.net/) for the new storage and CDN, and it looks to be excellent value, useful add-ins (for [image resizing and optimisation](https://docs.bunny.net/docs/stream-image-processing)), and most of all, PHP support - something that the site's usual hosting platform, Azure, [is not apparently not willing to provide](https://github.com/thephpleague/flysystem/issues/1680), with a PHP SDK to talk to the relevant services.

I was curious though - how easy would it be to add Bunny into a Symfony-based project, since it doesn't come included as a first-class storage adaptor in the [flysystem-bundle](https://github.com/thephpleague/flysystem-bundle).

Turns out - quite easily, since the adapter can be either one of the list of built-in adapters, ***or*** the name of a service where you've created a fully-configured adapter yourself.


```yaml
# config/packages/flysystem.yaml:
# Read the documentation at https://github.com/thephpleague/flysystem-bundle/blob/master/docs/1-getting-started.md
flysystem:
    storages:
        local.storage:
            adapter: 'local'
            options:
                directory: '%kernel.project_dir%/storage/default'

        uploads.storage:
            adapter: 'flySystem.uploads.adaptor'
```

There is also a couple of other options available to add here - `public_url_generator` & `temporary_url_generator` that refer to other pre-created, named services.

Here's how a custom-adaptor can be created, in `config/services.yaml`:
```yaml
services:
    flySystem.uploads.client:
        class: PlatformCommunity\Flysystem\BunnyCDN\BunnyCDNClient
        arguments:
            $storage_zone_name: 'zone-name'
            $api_key: 'api_key......'
            $region: 'uk'
    flySystem.uploads.adaptor:
        class: PlatformCommunity\Flysystem\BunnyCDN\BunnyCDNAdapter
        arguments:
            $client: '@flySystem.uploads.client'
            $pullzone_url: 'https://uploads.example.com'
```

Normally, the details for the `BunnyCDNClient`, and `$pullzone_url` in the `BunnyCDNAdapter` will come from an enviroment variable (and possibly decoded from a URL-like DSN).

And finally, the system can be used by typhinting `FilesystemOperator` & giving the name - here, `uploads.storage` converts to `$uploadsStorage`, and the argument is ready to use, for reading and writing files to the Bunny storage zone, and then via the CDN/pull-zone for general use.

```php
<?php
// src/Controller/FlySystemExampleController.php:
namespace App\Controller;
// use ...

#[Route('/example/fly')]
class FlySystemExampleController extends AbstractController
{
    public function __invoke(private FilesystemOperator $uploadsStorage) {
        dd($uploadsStorage);
    }
}
```

To get the other version of the FlySystem Filesystem that was configured in flysystem.yaml, just name it when you typehint the class:

```php
	public function __invoke(private FilesystemOperator $localStorage) {
        dd($localStorage);
    }
```
