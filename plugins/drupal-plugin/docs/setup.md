1. **Import content to your Drupal instance**

   For this implementation we used **Pantheon** as our host. So some configurations may be specific to that platform. Before importing the sql dump file we recommend extracting and adding the files located in **`data/files.zip`** to your drupal site under **`sites/default/`** or wherever your files folder is located on your instance. Afterwards you may use the **sql** dump file provided in the same **data** directory called **`homepage-starter-dump.sql.gz`**. Depending on the setup, you may have to extract the sql file before trying to import the data.

   ## Hosting on Pantheon

   1. Go to Pantheon.io, register and log in
   1. Create a new blank project and provide a name for the project

      <img src="./docs/images/setup-step-1.png" width="300">

   1. Select Drupal with Composer and then following the instructions to complete the installation

      <img src="./docs/images/setup-step-2.png" width="300">

   1. On the **Dashboard** there will be three (3) environments (_Dev, Test and Live_) and for our purposes we will use **Dev**. Select _Database/Files_ then _Wipe_. Click **_Wipe the Development Environment_** and follow the instructions to start with an empty site.

      <img src="./docs/images/setup-step-3.png" width="300">

   1. Go to _Import_. Here under **MySQL Database** select **File** and use the **homepage-starter-dump.sql.gz** provided in the data directory to upload the database. Make sure _Run update.php after the database import finishes_ is selected before uploading the file. Click **_Import_**.

      <img src="./docs/images/setup-step-4.png" width="500">

   1. Under **Archive of site files** select **File** and use the **files.zip** also provided in the data directory to upload the files. Click **_Import_**.

      <img src="./docs/images/setup-step-5.png" width="500">

   1. **_Clear Caches_** and and test out your site by clicking either **_Visit Development Site_** or **_Site Admin_**.

      <img src="./docs/images/setup-step-6.png" width="500">

   1. The credentials for logging in are:
      `sh username: admin password: DrupalGatsby123`
      It is highly recommended that you change the password to your Drupal site afterwards to something that only you know.

      Now, our site is up but we still need to install the [Gatsby Module](https://www.drupal.org/project/gatsby). To do that on **Pantheon** we need to pull down the site locally and install the module using **composer**. To streamline this process we will use a free, open source, cross-platform tool called [Lando](https://lando.dev/) which you can [Download Here](https://github.com/lando/lando/releases/tag/v3.6.0) (At the time v3.6.0 was the latest stable version).

### Lando & Pantheon Integration

For a video guided step-by-step tutorial see the links below:

[Local dev for Pantheon sites with Lando by Jantcu](https://www.youtube.com/watch?v=88WUuV-WJao)

1. Install **Lando** and **Docker**
1. A **Machine Token** is needed by **Pantheon** in order to _push and pull_ the **Database, Files and Code**. To generate a **Machine Token** follow these [instructions](https://pantheon.io/docs/machine-tokens). Remember that the **Machine Key** will only be visible once so keep it handy.
1. ```sh
   # Create a new directory for your Drupal site
   mkdir homepage-starter
   cd homepage-starter

   # Initialize Lando and when prompted select Pantheon and paste in the Machine Key generated earlier. Continue following the prompts provided to pull donw your site.
   lando init

   # Start server
   lando start

   # Pull down Database, Files and Code. We are working on the dev server so be sure to select "dev" when prompted
   lando pull

   # Clear caches
   lando drush cr
   ```

1. ```sh
   # Manually install modules
   lando composer config repositories.2 '{"type": "package", "package": { "name": "ionaru/easy-markdown-editor", "version": "2.15.0", "type": "drupal-library", "dist": { "url": "https://registry.npmjs.org/easymde/-/easymde-2.15.0.tgz", "type": "tar" } } }'
   lando composer require 'drupal/gatsby:^1.0@RC'
   lando composer require 'drupal/markdown:^3.0@RC'
   lando composer require 'league/commonmark:^1.0'
   lando composer require 'drupal/simplemde:^1.0@alpha'

   # Optional but makes navigation easier
   lando composer require 'drupal/admin_toolbar'

   # Clear caches again
   lando drush cr

   # Push up Database, Files and Code. We are working on the dev server so be sure to select "dev" when prompted
   lando push
   ```

All the modules should now be installed and activated. To ensure that they are all installed correctly:

1. Go to your local **Drupal** site and login.

1. Select _Extend_ in the toolbar.

   <img src="./docs/images/setup-step-7.png" width="500">

1. Find the **Gatsby Section** and check **_Gatsby_**, **_Gatsby Fast Builds_**, **_Gatsby JSON:API Instant Preview and Incremental Builds_**. All other dependent modules will automatically be installed.

   <img src="./docs/images/setup-step-8.png" width="500">

1. Find the **Web Services Section** and ensure that **_HTTP Basic Authentication_** is checked.

   <img src="./docs/images/setup-step-9.png" width="500">

1. Head to the bottom on the page and click the **Install** button.

1. Now you're done in your Drupal site! But we have one more step remaining to connect to your Gatsby homepage site.

1. **Run the setup script**

After setting up the Drupal site, navigate back to your Gatsby site's root directory and run:

```sh
yarn setup
```

This will run a script to create `.env.development` and `.env.production` files for you populated with your Drupal site environment variables.

---

## Local Drupal Development

The **composer.json** file as well as exported configurations found in the **config** folder are also included. If you decide to import and install these configurations, please do so before executing the **sql** script and be sure **not** to clean the existing database.

```sh
# import configurations
drush cim

# initial install
composer update

# installing from composer.lock
composer install
```

### Drush

For more information on how to use drush commands and how to install the command line shell visit [Drush Documentation Site](https://www.drush.org/latest/).

```sh
# If you wish to start from a clean site
drush sql-drop
drush sql-cli < ~/path/to/homepage-starter-dump.sql
```

An **admin** user already exists in the application. You will have to reset the password if you decide to start from a clean site.

```sh
# Drush 9
drush user:password admin "new_password"

# Drush 8 & earlier
drush user-password admin --password="new_password"
```

### Lando

A free, open source, cross-platform, local development environment and DevOps tool built on Docker container technology and developed by Tandem. [See the docs](https://docs.lando.dev/).

```sh
# This will destroy the database and import the data.
# If you wish to keep you existing data add the --no-wipe flag.
lando db-import ~/path/to/homepage-starter-dump.sql
```
