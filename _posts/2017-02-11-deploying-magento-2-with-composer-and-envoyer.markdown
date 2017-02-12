---
layout: post
title:  "Deploying Magento 2 with Composer and Envoyer"
date:   2017-02-11 20:43:45 -0400
categories: magento deployment envoyer
excerpt: In this post, I'll show you how to deploy Magento 2 the right way, using Composer and Envoyer, with zero(ish) downtime.
comments: true
---
Magento 2 offers significant improvements over Magento 1 in many areas, with one particular highlight being the ability to manage your code with [composer](https://getcomposer.org). If you're not familiar with composer, it's a dependency manager for PHP, meaning all of your code dependencies can be defined in a single JSON file, which results in a much lighter footprint when it comes to version control.

The caveat is that deploying sites managed with composer requires a little more configuration on the code deployment side, since we need to resolve the composer dependencies at deploy time (unless you're versioning the vendor folder, which you really shouldn't).

Being a huge fan of [Laravel](https://laravel.com), I found [Envoyer](https://envoyer.io), a zero downtime deployment tool for PHP applications. Since Magento 2 is a PHP-based application — and I'm really into composer — I decided to give it a go.

## Zero downtime deployments with Envoyer
So firstly, a quick overview of how Envoyer handles zero downtime deployments. On the face of it, it's actually quite simple — symlinks. Each time you deploy code with Envoyer, it creates a new folder on the server in a "releases" folder. The name of the folder is timestamped with the time of the deployment to keep it unique e.g. `releases/20170128235240`. Once a deployment is successful, Envoyer creates a "current" folder symlink to that newly created releases folder.

In order for everything to work, you'll need to set the document root on your server to point to this current folder.

_Note:_ If you're using cPanel (like me), cPanel doesn't like the document root to be changed. To get around this I simply created a `.htaccess` file at the server root, to rewrite the base URL as follows:
{% highlight apache linenos %}
RewriteEngine on
RewriteCond %{HTTP_HOST} ^mydomain.com$ [NC,OR]
RewriteCond %{HTTP_HOST} ^www.mydomain.com$
RewriteCond %{REQUEST_URI} !current/
RewriteRule (.*) /current/$1 [L]
{% endhighlight %}

## GIT Repository
In order to use Envoyer, your code must first be versioned with GIT, with a remote repository through [Bitbucket](http://bitbucket.org/), [GitHub](https://github.com) or [GitLab](https://gitlab.com). I happen to be using BitBucket, but it really doesn't matter since Envoyer gives you an easy interface to connect any one of them.

## Server Access
Envoyer pulls code directly from your connected repository onto your server, so you'll need to grant server access via a SSH key. My site is hosted with the fine folks at [Crucial Hosting](http://crucialhosting.com/) (which uses cPanel), so creating SSH access was a breeze. Once you have server access set up, Evoyer gives you the green light (literally!) in their interface so you know that everything is good to go.

## Deployment Hooks
This is where the magic happens! Envoyer gives you the ability to run deployment hooks before and after various stages of the deployment process, which are:

 1. Clone New Release
 2. Install Composer dependencies
 3. Activate New Release
 4. Purge Old Releases

### 1. Clone New Release
Each time you deploy code, Envoyer clones the code repository into a new folder on your server, keeping your current production code in tact.

### 2. Install Composer dependencies
This part is fairly explanatory. Envoyer automatically looks for your `composer.json` file, and runs `composer install`. For your first ever deploy this can take a few minutes — since Magento 2 has a lot of dependencies — but subsequent deployments will pull dependencies from cache, meaning they resolve in 10-20 seconds (depending on your server).

### 3. Activate New Release
Activating a new release means the current codebase symlink is updated to point at the new release directory.

### 4. Purge Old Releases
Envoyer keeps a copy of the last five or so releases, so this part just cleans up some of the older releases by removing them.

## Configuring Deployments hooks for Magento 2
Alright, so now we know what Envoyer can do, it's time to create specific deployment hooks for Magento 2. Here's the hooks I have:

### 1. Install Composer Dependencies (Before)
Envoyer will skip composer dependencies if a vendor directory already exists, or if the `composer.json` file is missing. In Magento 2, the vendor folder does exist, and is under version control since it contains a `.htaccess` file. In order for composer to run, we need to temporarily rename the vendor folder:

{% highlight powershell linenos %}
cd {% raw %}{{release}}{% endraw %}
mv vendor vendor_original
{% endhighlight %}

_Note_: the `{% raw %}{{release}}{% endraw %}` placeholder on line 1. This is a smart variable in Envoyer that gives the path to the current release that is being deployed. Pretty sweet!

### 2. Install Composer Depencendies (After)
Little bit of cleanup to do here: Move the `.htaccess` folder from `vendor_original` into the newly created `vendor` folder:

{% highlight powershell linenos %}
cd {% raw %}{{release}}{% endraw %}
cp vendor_original/.htaccess vendor/
rm -R vendor_original
{% endhighlight %}

### 3. Activate New Release (Before)
We have a number of things to do here, before we can activate the release.

#### Add `env.php` and `config.php` files
These files live in the `app/etc` folder of Magento 2, and contain environment specific code. The `env.php` file includes your encryption key, backend URL and database credentials, wheras `config.php` file contains module information. These files shouldn't be versioned, since they are specific to an environment, so I created a shared folder on the server, and uploaded the production files so they could be copied on deployment.

{% highlight powershell linenos %}
cp {% raw %}{{project}}{% endraw %}/shared/app/etc/env.php {% raw %}{{release}}{% endraw %}/app/etc/env.php
cp {% raw %}{{project}}{% endraw %}/shared/app/etc/config.php {% raw %}{{release}}{% endraw %}/app/etc/config.php
{% endhighlight %}

_Note:_ The `{% raw %}{{project}}{% endraw %}` placeholder gives the path to your project root, which is configured within Envoyer.

#### Magento setup
This is the most complicated step, and the step that will most likely take the longest in the deployment.

{% highlight powershell linenos %}
cd {% raw %}{{release}}{% endraw %}
php bin/magento setup:static-content:deploy en_US en_GB
php bin/magento setup:di:compile
php bin/magento maintenance:enable
php bin/magento setup:upgrade --keep-generated
php bin/magento maintenance:disable
{% endhighlight %}

 - **Line 1:** Nothing new here, we simply move into the folder for the code that is being deployed.
 - **Line 2:** Using the Magento CLI , we [generate all of the static content](http://devdocs.magento.com/guides/v2.1/config-guide/cli/config-cli-subcommands-static-view.html), which includes compilation of LESS files etc. _Note:_ You'll see that I am specifiying two languages: en_US and en_GB. If you're using different languages, be sure to add theme here.
 - **Line 3:** Again, using the Magento CLI, we setup and [compile all Dependency Injections](http://devdocs.magento.com/guides/v2.0/config-guide/cli/config-cli-subcommands-compiler.html).
 - **Line 4:** Enable Maintenance mode. Whaaaaaaaaat!? I thought you said this was zero downtime? Okay, it really should be, but it is important to do this, and here's why. When you run the `setup:upgrade` command on line 5, Magento is going to update the `setup_module` table in the database. Since this deployment has not been activated yet, we have a moment where we could have a potential mismatch between the code and what is in this table, which could cause an error. Essentially, this step just gives us an extra safety net.
 - **Line 5:** Run the upgrade script to update any module schemas in the database.
 - **Line 6:** Disable maintenance mode.

### 3. Activate New Release (After)
Our deployed code should now be live! One last step — clearing the Magento cache.

{% highlight powershell linenos %}
cd {% raw %}{{release}}{% endraw %}
php bin/magento cache:flush
{% endhighlight %}

## Linked folders
Almost there, but there's one (very important!) final step. Since we are deploying and resolving our dependencies on each deployment, what happens to user generated files (think product images), and sessions (if you're storing them in the file system)? It's a great question, and fortunately Envoyer has a solution — linked folders!

A linked folder allows you to easily create a symlink from a folder in your release to more permanent files on your server. Within the Envoyer interface, linked folders are managed through the "Deployment Hooks" tab, by clicking on the "Manage Linked Folders" button. In my case, I store sessions in the file system so I created two linked folders.

### Media Catalog Folder
Prior to deployments with Envoyer, I created a shared folder on the server and moved the entire catalog folder to it. On each deployment a symlink is then created from the catalog folder to the shared folder.

From: `pub/media/catalog` to: `shared/pub/media/catalog`

### Sessions
Sessions are also symlinked in the same way, using the shared folder. _Note:_ If you are using the database for sessions, you can ignore this step.

From: `var/session` to: `shared/var/session`

## Lift off!
If everything goes to plan, your code will be pushed to the server, all deployment hooks will run, and a symlink will be created from the "current" folder on your server to this release. If anything fails during the deployment, the whole deployment will fail, and your currently deployed code will remain in tact.

Alright, that's enough from me. Enjoy!
