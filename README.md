# MediaWiki Guide

## Overview

The objective of this document is to outline the steps to set up a minimal MediaWiki v1.43 instance with mySQL database, S3 for file storage, and Semantic MediaWiki (SMW) extension for semantic modelling. Steps to back up and restore the application are also provided.

General installation steps:

1. Install `mediawiki-core-1.43`.
1. Generate `LocalSettings.php` via MediaWiki web interface.
1. Install extensions, including AWS and SMW.

## Prerequisites

- php8.4 and composer
- mySQL
- S3

## Installation

1. Install `mediawiki-core-1.43`. Reference: [Manual:Installing MediaWiki](https://www.mediawiki.org/wiki/Manual:Installing_MediaWiki)
   1. Download the source code and dependencies.
    ```
    curl -O https://releases.wikimedia.org/mediawiki/1.43/mediawiki-core-1.43.1.tar.gz 
    tar -xzf mediawiki-core-1.43.1.tar.gz
    rm mediawiki-core-1.43.1.tar
    mv mediawiki-1.43.1 my_wiki
    cd my_wiki
    composer update --no-dev
    ```
   1. Download a [skin](https://www.mediawiki.org/wiki/Category:All_skins) into `my_wiki/skins/` directory. Common skins: MinervaNeue, MonoBook, [Timeless](https://www.mediawiki.org/wiki/Skin:Timeless), Vector.
   1. Generate `LocalSettings.php`:
      1. Serve the application: `composer serve`.
      1. In a browser, navigate to the application's root URL and follow the on-screen instructions to download the generated `LocalSettings.php` file.
      1. Move it to the root directory of the project.

1. Install AWS extension. Reference: [mediawiki-aws-s3 repo](https://github.com/edwardspec/mediawiki-aws-s3).
   1. Download source into `extensions` directory: `git clone --depth 1 https://github.com/edwardspec/mediawiki-aws-s3.git extensions/AWS`.
   1. Create the file `composer.local.json` with the following content:
    ```{json}
    {
      "extra": {
        "merge-plugin": {
          "include": [
            "extensions/AWS/composer.json"
          ]
        }
      }
    }
    ```
   1. `composer update --no-dev`.
   1. Enable the extension in `LocalSettings.php` and specify the credentials for S3 connection.
    ```
    wfLoadExtension( 'AWS' );
    $wgFileBackends['s3']['endpoint'] = 'https://12345.r2.cloudflarestorage.com';
    $wgAWSRegion = 'apac';
    $wgAWSBucketName = 'my-wiki';
    $wgAWSBucketDomain = 'example.com';
    $wgAWSCredentials = [
      'key' => '12345',
      'secret' => '12345',
      'token' => false
    ];
    ```
   1. Visit "Special:Version" page of the wiki to verify successful installation. Test upload a file to verify connection with S3.

1. Install SMW. Reference: [SMW's Help:Installation](https://www.semantic-mediawiki.org/wiki/Help:Installation/Quick_guide).
   1. Add the following property to `composer.local.json`.
    ```{json}
    {
      ...
      "require": {
        "mediawiki/semantic-media-wiki": "5.0.2"
      }
    }
    ```
   1. `composer update --no-dev`.
   1. Enable SMW by adding the following lines in `LocalSettings.php`.
   ```
   wfLoadExtension( 'SemanticMediaWiki' );
   enableSemantics( 'example.org' );
   ```
   1. Run MediaWiki's `update.php` script to update DB schema: `php maintenance/run.php update.php`.
   1. Visit "Special:Version" page of the wiki to verify successful installation.

## Back up & restore