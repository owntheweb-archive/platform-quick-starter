# platform-quick-starter
This is a quick starter guide to working with platform.sh hosted projects.

IMPORTANT: This document is a work-in-progress. Feedback is welcome.

## Official Docs

The official platform.sh docs are very helpful. Start here and read through it:

https://docs.platform.sh/

## Git Reading Further

http://rogerdudler.github.io/git-guide/

## Web Development: Getting Started

Most of these setup items are useful for local development, no matter the hosting solution.

### Mac Users: Install MAMP
Free version is fine to give access to the latest version of PHP in terminal without disrupting core Mac settings (which can make life bad).
https://www.mamp.info/en/downloads/

### Mac Users: Make MAMP's PHP the Mac Default PHP (much newer)
Edit user profile file:

`nano ~/.profile`

Add MAMP's PHP to the PATH variable on any line before `export PATH` (adjust "php7.1.1" to what actually lives there, make sure 7.0 or higher for Composer requirements):

`PATH="/Applications/MAMP/bin/php/php7.1.1/bin:$PATH"`

If you don't see an "export PATH", add this line to the bottom of the file:

`export PATH`

Completely close out of terminal and re-open or force terminal to re-read profile settings with:

`source ~/.profile`

Test php version (should NOT be 5.x, SHOULD be 7.x)

`php -v`

###Mac Users: Make MAMP's mysql useable in terminal as it is not available by default

Edit user profile file:

`nano ~/.profile`

Add MAMP's mysql to the PATH variable on any line before `export PATH`:

`PATH="/Applications/MAMP/Library/bin:$PATH"`

Completely close out of terminal and re-open or force terminal to re-read profile settings with:

`source ~/.profile`

Test php version (should NOT be 5.x, SHOULD be 7.x)

### SSH Access
Setup RSA Keys and sync with platform.sh
https://docs.platform.sh/development/ssh.html

Note: SSH access will not be fully available until hosted environments are redeployed. See "Cool Random Stuff" at the bottom after getting setup.

### Git
Install Git, a primary tool for working with and syncing hosting code with multiple developers. See top downloads section of this page:
https://git-scm.com/downloads

### Composer
Install composer, used in the Drupal 8 world (along with many other standardized php projects) to install modules to a local website with a few keystrokes in terminal:

Change to the home directory before proceeding:

`cd ~/`

Then proceed with instructions at:
https://getcomposer.org/download/

### Drush
Drush is handy for clearing Drupal cache via Terminal among many other tasks. For this startup guide, drush will be installed locally to the website. See setup instructions below. Full details:
http://docs.drush.org/en/8.x/

### rsync
A version of this is pre-installed on a Mac with no action required to start.

### platform CLI
Install the platform command line interface (CLI), which will let you join the cool kids and make hosting things happen with a few keystrokes. Most hosting tasks are developer centric with the most useful features found in the CLI.
https://docs.platform.sh/overview/cli.html

For ease of use, add platform CLI to user profile path:

`nano ~/.profile`

Add platform CLI to path on any line above `export PATH`

`PATH="/Users/YOURUSERNAME/.platformsh/bin:$PATH"`

Completely close out of terminal and re-open or force terminal to re-read profile settings with:

`source ~/.profile`

Test installation with:

`platform list`

## Download and Build Local Website

platform primarily works through Git with additional features available through the platform command. Use platform command to accelerate Git items and access databases (and more!).

Move to a directory where you will clone a website from a remote environment.

```
cd ~/Documents/
mkdir thewebsite.org
cd thewebsite.org
```

Get a list of projects that live at platform.sh

`platform projects`

Select the project and download, confirming folder name that will be created for it when prompted:

`platform get PROJECTIDHERE`

Build the project, installing all contributed libraries and modules beyond custom dev code:

```
cd FOLDERNAME
platform build
```

Install drush locally by running:

`composer require drush/drush`

Setup a database in MAMP at http://localhost:8888/phpMyAdmin/ . In this example, a database was created named platform_website_org.

Download the remote database. 

`platform db:dump`

Find the file that was downloaded using ls:

`ls`

Example file: "PROJECTIDHERE--master--app--dump.sql"

Create a local database to use for the site (skip if one is already created):

`mysql -u root -proot -e "create database platform_website_org";`

To check if the database exists:

`mysql -u root -proot -e "show databases";`

Then overwrite local database with (altering to taste first) with:

`mysql -u root -proot platform_website_org < PROJECTIDHERE--master--app--dump.sql`

Enable local configuration settings by copying the local example settings file. Note: This will not be available for Git inclusion (pre-excluded for local flexibility and security):

`cp web/sites/example.settings.local.php web/sites/default/settings.local.php`

Add the following to the bottom of settings.local.php to configure local db settings and add a required hash, editing to taste:

```
/*
 * hash salt generally provided at runtime on platform.sh
 */

$settings['hash_salt'] = 'juXLp+xMCBiCrCTA*h6DVpveiXn4VH.2Z';

/*
 * Database configuration
 */

$databases['default']['default'] = array (
  'database' => 'platform_website_org',
  'username' => 'root',
  'password' => 'root',
  'host' => 'localhost',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
  'collation' => 'utf8mb4_general_ci',
);
```

Sync the Drupal files directory (image uploads, cache files, other non-dev code items):

`vendor/bin/drush rsync @PROJECTIDHERE.BRANCHNAME:%files @PROJECTIDHERE._local:sites/default`

## Creating A New Feature: Workflow

NOTE: The following is subject to change. We may want to add a staging environment before moving to live site.

Instead of thinking about Git for "dev", "test" and "live" environments (like Pantheon hosting), Chris recommends orientating git branches/environments towards features, so "boostrap-install", "advanced-custom-search", "master". That way one can switch back and forth as requests come up throughout the day, creating, checking out specific "topics" of development. It makes for flexible time saving code management. This page talks towards that style of Git management:
https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging

Work with a new feature branch:

`cd /local-website-dir`

Make sure you are on the master branch locally before attempting to branch it into a new feature branch (unless you want to branch from another branch):

`git checkout master`

Pull the latest and greatest from remote current environment

`git pull`

Tell platform to create a new branch of the master environment. This will also auto-run `git branch your-feature` locally and check it out/set as active branch (`git checkout twitter-bootstrap-install` otherwise if just returning to make continued edits).

`platform e:branch twitter-bootstrap-install`

Make some worthy file changes. In this example, I installed twitter bootstrap theme using composer. This could also be done locally by logging into the site and adding via the Drupal site admin area or by visiting the Drupal site, locating a module, downloading, extracting files where they belong (but the following one liner saves time):

`composer require drupal/bootstrap`

To see what changes were made that need to be committed, the following command may be helpful:

`git diff`

git add commands could be executed for each file individually, or all added in one go with:

`git add .`

commit changes with helpful message:

`git commit -m "Installed Twitter Bootstrap"`

Push the changes to a new environment on platform.sh. Note: New environments are charged at $21/3 environments. **CHECK THIS**: Chris thinks this is prorated though, not incurring much cost if a quick test and merge that removed the branch/environment when finished.

`git push`

Check out the site once the environment has been re-built and launched. To list available URLs for the current environment, type:

`platform e:url`

It will list a few. Type 0 and hit return to start. It will launch the website in the browser. This can also be found in the online admin interface if preferred.

## Merge Feature Branch to Master Branch (live site)

IMPORTANT: We may create a staging/testing branch in the near future to ensure all is good before going to the live site. This may or may not be needed. Standby for updates.

If the feature is complete, approved and ready to merge with the parent branch, which also eliminates the extra environment, do this:

`platform e:merge`

It will ask if you really want to proceed. If this is the case, type in 'Y' and hit return.

Note: As with most hosting environments, code updates like this will not affect the parent environment database. Any database configurations and content (usually set when logged into Drupal) will need to be re-applied there.

IF and ONLY IF you no longer need the feature environment, delete it to save on costs. We pay $21/month for three environments (prorated Chris thinks).

First, make sure to locally checkout the environment that needs to be deleted. Example:

'git checkout twitter-bootstrap-install'

Then, use platform to delete the remote environment.

`platform e:delete`

You'll still have the local branch. You can switch to master locally next:

`git checkout master`

Also as good habit:

`git pull`

## Cool Random Stuff

### Force Re-Deploy Environment

This will force a platform.sh environment to be redeployed, handy if new ssh keys are added or other non-git type account changes (NO environment files will change unless redeployed)

```
git commit --allow-empty -m 'force redeploy'
git push PROJECTIDHERE@git.us.platform.sh:PROJECTIDHERE.git master
```

### platform CLI fun

Open the hosting admin website

`platorm web`

### Composer: Update All Modules and Core with One Line!

`composer update`

Then:

```
git add composer.lock
git commit -m "Updated all the things with one line!
git pull
git push
```