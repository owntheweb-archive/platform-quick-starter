# Platform.sh Quick Starter Guide
This is a quick starter guide to working with platform.sh hosted projects. At the moment this is focused on Mac users deploying Drupal projects.

## Important Notes

This document is not endorsed platform.sh team gurus. It's a work-in-progress. Feedback and contributions are welcome.

## Official Documentation

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

If not seeing an "export PATH", add this line to the bottom of the file:

`export PATH`

Completely close out of terminal and re-open or force terminal to re-read profile settings with:

`source ~/.profile`

Test php version (should NOT be 5.x, SHOULD be 7.x)

`php -v`

### Mac Users: Make MAMP's mysql useable in terminal as it is not available by default

Edit user profile file:

`nano ~/.profile`

Add MAMP's mysql to the PATH variable on any line before `export PATH`:

`PATH="/Applications/MAMP/Library/bin:$PATH"`

Completely close out of terminal and re-open or force terminal to re-read profile settings with:

`source ~/.profile`

Test:

`mysql`

It should print out general information. Type `quit` and return to exit.

### SSH Access
Setup RSA Keys and sync with platform.sh
https://docs.platform.sh/development/ssh.html

Note: SSH access will not be fully available until hosted environments are redeployed. See ["Force Redeploy Environment"](#forceredeploy) after setup for details.

### Git
Install Git, a primary tool for working with and syncing hosted code with multiple developers. See top downloads section of this page:
https://git-scm.com/downloads

### Composer
Install composer, used in the Drupal 8 world (along with many other standardized php projects) to install modules and libraries to a local website via terminal. Change to the home directory before proceeding:

`cd ~/`

Then proceed with instructions at:
https://getcomposer.org/download/

### Drush
Drush is handy for clearing Drupal cache via Terminal among many other tasks such as syncing non-Git files. Great news: Drush is pre-installed with any local platform website install. More on Drush:
http://docs.drush.org/en/8.x/

### rsync
A version of this is pre-installed on a Mac with no action required to start.

### platform CLI
Install the platform command line interface (CLI), joining the cool kids and make hosting things happen with a few keystrokes. Most hosting tasks are developer centric with the most useful features found in the CLI (not the web admin UI).
https://docs.platform.sh/overview/cli.html

For ease of use, add platform CLI to user profile path:

`nano ~/.profile`

Add platform CLI to path on any line above `export PATH`

`PATH="/Users/USERNAME/.platformsh/bin:$PATH"`

Completely close out of terminal and re-open or force terminal to re-read profile settings with:

`source ~/.profile`

Test installation with:

`platform list`

## Download and Build Local Website

platform primarily works through Git with several additional features available through the platform command. Use platform to accelerate Git items, access databases and more!

Move to a directory where cloned websites from remote environments will live locally. For example:

```
cd ~/Documents
```

Get a list of projects that live at platform.sh

`platform projects`

Select the project and download, confirming folder name that will be created for it when prompted:

`platform get PROJECTIDHERE`

Build the project, installing all contributed libraries and modules outside of custom code stored in the Git repository:

```
cd FOLDERNAME
platform build
```

Download the remote database. 

`platform db:dump`

Find the file that was downloaded using ls:

`ls`

Example file: "PROJECTIDHERE--master--app--dump.sql"

Create a local database to use for the site:

`mysql -u root -proot -e "create database platform_website_org";`

Confirm that the database exists:

`mysql -u root -proot -e "show databases";`

Then overwrite local database with (altering to taste first):

`mysql -u root -proot platform_website_org < PROJECTIDHERE--master--app--dump.sql`

Enable local configuration settings by copying the local example settings file. This will not be available for Git inclusion (pre-excluded for local flexibility and security). Be aware that the example.settings.local.php file includes various flags and toggles to force Drupal into debug mode. That's a good thing a majority of the time. Configuration can also be altered as needed:

`cp web/sites/example.settings.local.php web/sites/default/settings.local.php`

Add the following to the bottom of settings.local.php to add a required hash (supplied by platform when remote, not local) and configure local db settings. Edit as needed:

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

First, move to the project folder:

`cd /local-website-dir`

Checkout the master branch locally before attempting to branch it into a new feature branch (unless planning to branch from another branch):

`git checkout master`

Pull the latest code from the current remote repository:

`git pull`

Tell platform to create a new branch of the current remote repository via platform. This will also auto-run `git branch twitter-bootstrap-install` locally and check it out/set as active branch (`git checkout twitter-bootstrap-install`).

`platform e:branch twitter-bootstrap-install`

If the branch was already created locally before running the above line, the following example steps would activate a platform environment:

```
git checkout twitter-bootstrap-install
git push
platform e:activate

```

Make a few worthy file changes. In this example, Twitter Bootstrap theme was installed using composer. This could also be completed locally by logging into the site and adding via the Drupal site admin area or by visiting the Drupal site, locating a module, downloading, extracting files where they belong (but the following one liner saves time and works better with great platform Composer features, less code to commit):

`composer require drupal/bootstrap`

To see what changes were made that need to be committed, the following command may be helpful:

`git diff`

Git add commands could be executed for each file individually, or all added all at once with:

`git add .`

For a more interactive approach with detailed options of what/what not to stage:

`git add -p`

Commit changes with helpful message:

`git commit -m "Installed Twitter Bootstrap"`

Push the changes to the repository on platform.sh. Durring this process, an updated environment is launched to show code changes, replacing the previous environment once launched without downtime (continuous deployment).

`git push`

Check out the site once the environment has been rebuilt and launched. To list available URLs for the current environment, type:

`platform e:url`

A few URLs will be listed. Type 0 and hit return to start. It will launch the environment's website in the default browser. This can also be found in the online admin interface if preferred.

## Make a Snapshot (backup)

Before merging the feature branch into the parent branch, it may be a good idea to make a backup of the parent environment which includes files and database. Snapshots are stored on Platform for 7 days and are helpful in case something goes wrong.

Checkout the parent branch associated with the environment that needs a snapshot, e.g.:

`git checkout master`

Make a snapshot:

`platform snapshot:create`

Return to the Git feature branch that will be merged:

`git checkout twitter-bootstrap-install`

## Merge Feature Branch to Master (Live) or Parent Branch

Once the feature is complete, approved and ready to merge with the parent branch, initiate a platform merge. This will also redeploy the parent environment (without downtime):

`platform e:merge`

Type in 'Y' and hit return to confirm the merge.

Note about databases: As with most hosting solutions and development workflows, Git-related code updates will not affect the parent environment database (or any database). Database configurations and content (usually set when logged into Drupal) will need to be reapplied to the parent environment website.

## Delete Feature platform Environment

Make sure to locally checkout the branch that has a remote environment that needs to be deleted. Example:

'git checkout twitter-bootstrap-install'

Then, use platform to delete the remote environment. This will not delete the remote or local Git branch (if adding `--no-delete-branch` option), only the running environment to save on available hosting plan resources (can be reactivated anytime).

`platform e:delete --no-delete-branch`

The local branch will still be checked out. Switch to master locally with:

`git checkout master`

Also a good habit to ensure all is up-to-date locally before proceeding with another feature branch, etc.:

`git pull`

### Activate Environment for Existing Offline Remote Branch

If a Platform environment has been deleted and needs reactivated and/or any remote repository offline branch needs to have an active environment, checkout the local branch and activate with:

```
git checkout the-feature-branch
platform e:activate
```

## Cool Random Stuff

### <a name="forceredeploy"></a>Force Redeploy Environment

This will force a platform.sh environment to be redeployed, handy if new ssh keys are added or other non-git type account changes (NO environment files will change unless redeployed, a unique platform feature)

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
git commit -m "Updated all the things with one line!"
git pull
git push
```

### Prevent Database Dumps From Getting Committed

todo: add to .gitignore

## Todo

- Include a note about sync (haven't tried it yet, heard it's cool)
- Add more "Cool Random Stuff"
- Everything else that is being forgotten...
