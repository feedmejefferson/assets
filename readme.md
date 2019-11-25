# Feed Me Assets

> AKA the meta data backup repository

This repo maintains a backup of the feedme-moderator firebase data (both firestore and cloud storage blob files), which constitutes a superset of the more dynamic (managed) data that we publish for Feed Me, Jefferson! This is separate from the brand assets managed in the `feedme-branding` repository. 

## Purpose(s)

This repository serves a few purposes:

1. backup curated/moderated metadata so that we can rollback to earlier sync points
2. provide an offline copy of the metadata for analytics purposes (see [feedme-data](https://github.com/feedmejefferson/feedme-data)
3. host a copy of the assets as static pages via github pages to support some of our [data visualizers](https://feedmejefferson.github.io/feedme-data/)

## Structure 

In order to work with github pages, we're burrying all of the managed assets that we want to host in the [docs](docs) folder. Any scripts, tools or configuration files should live outside of that scope.

The [docs/meta](docs/meta) folder contains exported firestore data collections.

The [docs/theme](docs/theme) folder may contain some minimal branding assets for styling various github pages project sites.

All other folders in the [docs](docs) folder contain cloud storage assets that should be synced to the `assets` folder of cloud storage.

We use the [reindexed](reindexed) folder as a staging folder for files coming back from the R script. These still need to be preprocessed by the `ladle associate` command before being imported back into the firestore `indices` collection.

## Tooling

Ladle is a very rough command line tool built specifically to assist us with importing and exporting our dynamic firestore data. We do this both for backing up the data (which gives us the ability to roll back to previously saved backups by reimporting the data) but also for offline analytic processing and indexing. 

The exported data supports a number of scripts in the `feedme-data` project. 


## Installation

### Install `gsutil`

[gsutil installation docs](https://cloud.google.com/storage/docs/gsutil_install)

### Install Ladle

    npm install
    
### Install firebase Service Account Key

Get the json file for a working service account key and download/copy it into this directory with the file name `serviceAccountKey.json`. 

> Make sure never to commit it to the git repo for the world to see. That filename should already be in the `.gitignore` file. 

## Firestore Export / Indexing Pipeline

### Run the Export

```
npx ladle export foods database/foods
```

### Run the feedme-data index-foods.R script

### Preprocess the files

The R script creates an array of objects. We need to convert that to an associative array (aka a map object) wrapped inside of the `data` field of another object. The ladle associate command does that for us.

```
npx ladle associate reindexed/tagStats.json docs/meta/indices/tagStats
npx ladle associate reindexed/foodStats.json docs/meta/indices/foodStats
npx ladle associate reindexed/tagFoods.json docs/meta/indices/tagFoods
```

### Import the update Indexes

```
npx ladle import database/indices indices
```

## Cloud Storage

### Import Assets

    gsutil -m rsync -x 'theme/|meta/' -r -a public-read docs gs://feedme-moderator.appspot.com/assets

### Export Assets

    gsutil -m rsync -r gs://feedme-moderator.appspot.com/assets docs
