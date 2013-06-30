# parallel-flickr

https://github.com/straup/parallel-flickr/blob/one-by-one/README.DRAFT.md#backing-up-photos

https://github.com/straup/parallel-flickr/blob/one-by-one/README.DRAFT.md#storage-options

_This is a straight-up clone of @vicchi 's [documentation for privatesquare](https://github.com/straup/privatesquare/blob/master/README.md).

## Gentle Introduction

parallel-flickr is a tool for backing up your Flickr photos and generating a database backed website that honours the viewing permissions you've chosen on Flickr. parallel-flickr is **not** a replacement for Flickr.

It uses the [Flickr API](http://www.flickr.com/services/api) as a single-sign-on
provider, for user accounts, and to [WORDS]

parallel-flickr is still a work in progress. It is more than an ideal research
project. It is working code that I use every day. On the other hand it is also
not a full-time gig so I work on it during the mornings and the margins of the
day so it is not pretty or classy yet and does not have a one-button
installation process but it _does_ work.

**It almost certainly still contains bugs, some of them really stupid.**

In the meantime, [here's a blog post](http://www.aaronland.info/weblog/2011/10/14/pixelspace/#parallel-flickr) and [some screenshots](http://www.flickr.com/photos/straup/tags/parallelflickr/).


## Installation - The Short Version

parallel-flickr is built on top of [Flamework](https://github.com/exflickr/flamework) which means it's nothing more than a vanilla Apache + PHP + MySQL application. You can run it as a dedicated virtual host or as a subdirectory of an existing host.

You will need to make a copy of the [config.php.example](https://github.com/straup/parallel-flickr/blob/master/www/include/config.php.example) file and name it `config.php`. You will need to update this new file and add the various specifics for databases and third-party APIs.

	# You will need valid Flickr API credentials
	# See also: http://www.flickr.com/services/apps/create/apply/

	$GLOBALS['cfg']['flickr_api_key'] = '';
	$GLOBALS['cfg']['flickr_api_secret'] = '';	

	$GLOBALS['cfg']['flickr_api_perms'] = 'read';

	# You will need to setup a MySQL database and plug in the specifics
	# here: https://github.com/straup/parallel-flickr/blob/master/schema
	# See also: https://github.com/straup/flamework-tools/blob/master/bin/setup-db.sh

	$GLOBALS['cfg']['db_main'] = array(
		'host' => 'localhost',
		'name' => 'parallel-flickr',
		'user' => 'parallel-flickr',
		'pass' => '',
		'auto_connect' => 1,
	);

	# You will need to set up secrets for the various parts of the site
	# that need to be encrypted. Don't leave these empty. Really.
	# You can create new secrets by typing `make secret`.
	# See also: https://github.com/straup/parallel-flickr/blob/master/bin/generate_secret.php

	$GLOBALS['cfg']['crypto_cookie_secret'] = '';
	$GLOBALS['cfg']['crypto_crumb_secret'] = '';
	$GLOBALS['cfg']['crypto_password_secret'] = '';

	# If you don't have memcache installed (or don't even know what that means)
	# just leave this blank. Otherwise change the 'cache_remote_engine' to
	# 'memcache'.

	$GLOBALS['cfg']['cache_remote_engine'] = '';
	$GLOBALS['cfg']['memcache_host'] = 'localhost';
	$GLOBALS['cfg']['memcache_port'] = '11211';

	# This is only relevant if are running parallel-flickr on a machine where you
	# can not make the www/templates_c folder writeable by the web server. If that's
	# the case set this to 0 but understand that you'll need to pre-compile all
	# of your templates before they can be used by the site.
	# See also: https://github.com/straup/parallel-flickr/blob/master/bin/compile-templates.php

	$GLOBALS['cfg']['smarty_compile'] = 1;

## Installation - The Long, Slightly Hand Holding, Version

Get the [code from GitHub](https://github.com/straup/parallel-flickr).

Decide on whether you'll host this on a sub-domain (something along the lines of `parallel-flickr.example.com`) or in a subdirectory (maybe something like `www.example.com/parallel-flickr`).

This rest of this section will assume the following:

* That you'll be hosting on a subdomain called *parallel-flickr* on a domain called *example.com*, or, to put it another way `parallel-flickr.example.com`. Just mentally substitute your domain and sub-domain when reading, and physically substitute your domain and sub-domain during the installation process. Unless you actually own the example.com.
* That you'll be using Flickr for reverse-geocoding and not an instance of the `reverse-geoplanet` web-service.
* That you want the URL for parallel-flickr to be `parallel-flickr.example.com` and not `parallel-flickr.example.com/www`
* That you want parallel-flickr to be on a public facing web service. You *can* install it on a local machine that isn't publicly accessible but to do this needs some careful copying-and-pasting of database settings from a public facing machine to your local, private machine. See the *Here-Be-Dragons Locally Hosted Version Below* if you want to get your hands dirty.
* That `<root>` is the path on your webserver where your web server has been configured to find the sub-domain.
* That you have shell access (probably via SSH) to your web server.

Register with Flickr - go to http://www.flickr.com/services/apps/create/apply/

* Apply for a non-commercial key
* Set the *App name* to `parallel-flickr` (or something that's meaningful to you)
* Set the *App description* to something meaningful, such as *An instance of https://github.com/straup/parallel-flickr*
* Tick both boxes!
* Note the *API key* and *API secret* the registration process gives you (it's a good idea to save this in a new browser window or tab so you can copy-and-paste)

Now ... upload the code, plus all sub-directories to your web-server; don't forget the (hidden) `.htaccess` file in the root of the code's distribution.

Copy `<root>/www/include/config.php.example` to `<root>/www/include/config.php` and edit this new file.

Copy-and-paste your Flickr Key into the section of the config file that looks like ...

	$GLOBALS['cfg']['flickr_api_key'] = 'my-flickr-key-copied-in-here';
	$GLOBALS['cfg']['flickr_api_secret'] = 'my-flickr-secret-copied-in-here';	

By default WORDS ABOUT PERMISSIONS

	$GLOBALS['cfg']['flickr_api_perms'] = 'read';
	
Set up your database name, database user and database password. Copy and paste these into ...

	$GLOBALS['cfg']['db_main'] = array(
		'host' => 'localhost',
		'name' => 'my-database-name',
		'user' => 'my-database-user',
		'pass' => 'my-database-users-password',
		'auto_connect' => 0,
	);

Setup your encryption secrets secrets. SSH to your host and run `php <root>/bin/generate_secret.php`, 3 times. Copy and paste each secret into 

	$GLOBALS['cfg']['crypto_cookie_secret'] = 'first-secret-here';
	$GLOBALS['cfg']['crypto_crumb_secret'] = 'second-secret-here';
	$GLOBALS['cfg']['crypto_password_secret'] = 'third-secret-here';

(If you don't have shell access to your web-server, you can run this command from the shell on a local machine)

Create the database tables. Load `<root>/schema/db_main.schema`, `<root>/schema/db_tickets.schema` and `<root>/schema/db_users.schema` into the database. You can do this either via phpMyAdmin and the import option or via `mysql` on the shell's command line

Browse to http://parallel-flickr.example.com

If you get errors in your Apache error log such as ...

	www/.htaccess: Invalid command 'php_value', perhaps misspelled or defined by a module not included in the server configuration

... then your host is probably running PHP as a CGI and not as a module so you'll want to comment out any line in `<root>/www/.htaccess` that starts with `php_value` or `php_flag` and put these values into a new file, `<root>/www/php.ini`, without the leading `php_value` or `php_flag`.

Click on *sign in w/ flickr* and authenticate with Flickr.

WORDS ABOUT BACKUPS

You might want to put this command in a cron job, if your web host allows this.

That's it. Or should be. If I've forgotten something please let me know or submit a pull request.

## Installation - The Here-Be-Dragons Locally Hosted Version

If you really want to hack and play around with parallel-flickr, it's best to do
this on a private, locally hosted machine, like your laptop or your desktop
machine. But as a starting point you need to have followed the installation
instructions above as you *need* to have a public facing installation first and
then clone this to your local machine. The reason for this is that you need to
authenticate with Flickr, Flickr uses FlickrAuth or OAuth to authenticate and OAuth authentication needs a publicly accessible web server to authenticate *with*. With that said, roll up your sleeves, grab a cup of your caffeinated beverage of choice and follow along.

This rest of this section will assume the following:

* That you're running [MAMP](http://mamp.info/en/index.html) on a Mac. MAMP is a nice convenient way to run MySQL, Apache and PHP on a Mac. There's also a Windows version called [WAMP](http://www.wampserver.com/en/). Or most Linux distros come with all of this installed. YMMV so you may need to change some paths and file names.
* That you'll set up a local host name called `parallel-flickr`
* That your MAMP installation is running Apache on port 8888 and MySQL on port 8889.

So ... firstly create a local host name by adding `parallel-flickr` to your `/etc/hosts` file, which will look something like this ...

	127.0.0.1	localhost parallel-flickr

On some operating systems, this file is re-read each time your browser is re-started. On Mac OS X, you'll also need to flush and reload the machine's DNS cache ...

	$ sudo /usr/bin/dscacheutil -flushcache

Now create a new virtual host on your machine. Edit `/Applications/MAMP/conf/apache/httpd-vhosts.conf` and append the magic incantation ...

	<VirtualHost *:8888>
		ServerName parallel-flickr:8888
		DocumentRoot "/Applications/MAMP/htdocs/parallel-flickr/www"
	</VirtualHost>

Restart Apache in MAMP. Create `/Applications/MAMP/htdocs/parallel-flickr/www`. Browse to `http://parallel-flickr:8888`. Check you get an empty directory listing of / to ensure your virtual host is configured and working correctly.

Now download your working, public, parallel-flickr install (to ensure any customisations, including configuration, you've made are preserved) from your public facing webserver.

Export your parallel-flickr database from your public facing installation, either via phpMyAdmin / export or via the `mysqldump command` from the shell's command line.

Create a new local database to hold the, err, data.

Import your parallel-flickr database to your local installation, either via phpMyAdmin / import or via the `mysqldump` command from the shell's command line.

Edit your local copy of parallel-flickr's configuration file at `/Applications/MAMP/htdocs/parallel-flickr/www/include/config.php` to point to your new local database.

	$GLOBALS['cfg']['db_main'] = array(
		'host' => 'localhost',
		'name' => 'parallel-flickr',
		'user' => 'root',
		'pass' => 'root',
		'auto_connect' => 0,
		);

Still in the local parallel-flickr configuration file, set the `environment` config value to be `localhost`:

	$GLOBALS['cfg']['environment'] = 'localhost';

Now browse back to `http://parallel-flickr:8888`. You should be asked to *sign in w/ flickr*. Don't. You'll be redirected to the Flickr site to authenticate and this will fail as your local install isn't publicly accessible.

WORDS ABOUT BACKUPS

	PATH=/Applications/MAMP/Library/bin:/Applications/MAMP/bin/php/php5.3.6/bin:$PATH
	export $PATH

## Backing up photos

After setting up everything above, and setting your API key callback to "http://YOURDOMAINNAME.com/auth/", visit /account/backups/. This will
create your backup user account and then from here you can run the various backup scripts inside of the bin/ directory. 

### Backing up photos manually

It is helpful to set these various bin/backup_* scripts to run via cron. According to your level of faving, uploading, and contacts fiddling, you may have your own requirements for often you want to run the various backup scripts.

Here's a once-a-day example, which works for a moderate level of activity:

    0 3 * * * php /full/path/to/parallel-flickr/bin/backup_contacts.php
    15 3 * * * php /full/path/to/parallel-flickr/bin/backup_faves.php
    30 3 * * * php /full/path/to/parallel-flickr/bin/backup_photos.php

### Automagic backing up of your photos (using the Flickr PuSH feeds)

parallel-flickr can also be configured to archive the photos for registered users
using the [real-time photo update PuSH feeds](http://code.flickr.com/blog/2011/06/30/dont-be-so-pushy/)
from Flickr.

By default this functionality is disabled  default because in order to use it
you need to ensure that the directory specified in the
`$GLOBALS['cfg']['flickrstatic_path']` config variable is writeable by the web
server _or_ you need to run "storagemaster" storage provider, described below.

_Another way to deal with the probem of permissions is just to use Amazon's S3 service to store your photos, described below._

To enable the PuSH features you'll need to update the following settings in your
config file: 

	$GLOBALS['cfg']['enable_feature_flickr_push'] = 1;
	$GLOBALS['cfg']['enable_feature_flickr_push_backups'] = 1;	

The easiest way to enable PuSH backups is to go to the Flickr backups account
page (on your version of parallel-flickr). That is:

	http://your-website.com/account/flickr/backups/

If you've never setup backups before and the `flickr_push` configs described
above have been enabled then PuSH backups will be enabled at the same time that
backups (for your photos, your faves, etc.) are registered.

Not all backup types are valid PuSH backup types (like your contact list, for
example).

If you have enabled "poor man's god auth"
[in the config file](https://github.com/straup/parallel-flickr/blob/master/www/include/config.php.example) 
 (described above) then there is also a "god" page that will list all the PuSH subscription
registered for user backups and other features at this URL:

	http://your-website.com/god/push/subscriptions/

From here you can create or delete individual PuSH feeds, although the tools are
still feature incomplete. Specifically, it is not yet possible to register new
feeds with arguments (like a tag or a user ID).

## Storage options

Internally parallel-flickr uses an abstraction layer for storing (and
retrieving) files. This allows for a variety of storage "providers" to be used
with parallel-flickr. As of this writing they are:

* **fs** - files are read from and written to the local file system. This is the default provider.

* **s3** - files are read from and written to the Amazon Web Service (AWS) S3 service. You will need
	to include your AWS credentials in the `config.php` file in order for this
	provider to work.
	
* **storagemaster** - files are read from and written to a "storagemaster" daemon
    which is configured to run on a high-numbered port on the same machine as
    parallel-flickr itself. By default files are read from and written to the
    local file system with the important distinction that the storagemaster
    daemon itself is running as the `www-data` user (or whatever user account
    the Apache web server is using). Storagemaster details are discussed below.

	$GLOBALS['cfg']['storage_provider'] = 'fs';
	
### Using Amazon's S3 service for storing photos (and metadata files)

parallel-flickr is able to store your photos and metadata files using Amazon's
S3 storage service.

Setting up an Amazon account and getting an Amazon Web Services (AWS) API key
and secret are out of scope for this document (there are lots of good howtos on
Amazon's own site and the Internet at large) but once you do it's easily to
configure parallel-flickr to use S3. Specifically, you just need to add the
following settings to `config.php` file:

	$GLOBALS['cfg']['amazon_s3_access_key'] = 'YER_AWS_ACCESS_KEY';
	$GLOBALS['cfg']['amazon_s3_secret_key'] = 'YER_AWS_SECRET_KEY';
	$GLOBALS['cfg']['amazon_s3_bucket_name'] = 'A_NAME_LIKE_MY_FLICKR_PHOTOS';

### Using the "storagemaster" service for storing photos (and metadata files)

Storagemaster is a very simple socket-based daemon meant to run on a
high-numbered port on the same machine as parallel-flickr itself. That's not a
requirement but the important thing to remember is that it is **not** meant to
generally accessible on the public internet because it has no security controls
of its own.

Storagemaster exposes a deliberately simple interface for manipulating files:
EXISTS, GET, PUT and DELETE.

By default files are read from and written to the local file system with the
important distinction that the storagemaster daemon itself is running as the
`www-data` user (or whatever user account the Apache web server is using).

	# Storagemaster. See also:
	# parallel-flickr/storagemaster/bin/storagemaster.py
	# parallel-flickr/storagemaster/init.d/storagemaster.sh

The default `config.php.example` file which you've copied in to your instance of
parallel-flickr should already contain the following details, which you may want
to adjust to taste:

	$GLOBALS['cfg']['storage_storagemaster_host'] = '127.0.0.1';
	$GLOBALS['cfg']['storage_storagemaster_port'] = '9999';

If you do remember to update the `init.d/storagemaster.sh` file accordingly.

## Feature flags

Please write me

## The fancy stuff

### Uploads

	$GLOBALS['cfg']['enable_feature_uploads'] = 1;
	$GLOBALS['cfg']['enable_feature_uploads_archive'] = 1;
	$GLOBALS['cfg']['enable_feature_uploads_shoutout'] = 1;

### Upload by email 

	$GLOBALS['cfg']['enable_feature_uploads_by_email'] = 1;
	$GLOBALS['cfg']['uploads_by_email_hostname'] = 'upload.parallel-flickr.example.com';

	# 1048576 is 1MB; web based upload sizes are controled by the
	# 'upload_max_filesize' directive in www/.htaccess

	$GLOBALS['cfg']['uploads_by_email_maxbytes'] = 1048576 * 5;	

### Filt(e)ring uploads

	$GLOBALS['cfg']['enable_feature_uploads_filtr'] = 1;

	$GLOBALS['cfg']['filtr_valid_filtrs'] = array(
		'dazd',
		'dthr',
		'postr',
		'pxl',
		'pxldthr',
		'rockstr',
	);

### Solr

	$GLOBALS['cfg']['enable_feature_solr'] = 1;
	$GLOBALS['cfg']['solr_endpoint'] = 'http://localhost:8985/solr/parallel-flickr/';

	$GLOBALS['cfg']['enable_feature_places'] = 1;
	$GLOBALS['cfg']['places_prefetch_data'] = 1;

	$GLOBALS['cfg']['enable_feature_cameras'] = 1;
	$GLOBALS['cfg']['enable_feature_archives'] = 1;

### API

## TO DO:

In no particular order (patches are welcome):

* Make sure video files are actually being fetched properly

* Dets, galleries, groups

* People tagging (http://www.flickr.com/services/api/flickr.photos.people.getList.html)

* Dates and timezones... sad face

* Photo deletion

* Account deletion

* Context-specific URLs (e.g. in-faves or in-WOEID)

* Display metadata

* Search

* Better layout, tested in more than just Firefox

* Send to Internet Archive (http://www.archive.org/help/abouts3.txt)

See also: [TODO.txt](https://github.com/straup/parallel-flickr/blob/master/TODO.txt)

## Notes

### A note about (Github) branches:

If you look carefully you may see that there are a lot branches for
parallel-flickr in my Github repository. These are there purely (and only) for
my working purposes.

You're welcome to poke at them obviously but the rule of thumb is: If it's in
"master" then it should work, modulo any outstanding bugs. If it's in any other
branch then all the usual caveats apply, your mileage may vary and we offer no
guarantees or refunds.

## See also:

* [flamework](https://github.com/straup/flamework)

* [flamework-flickrapp](https://github.com/straup/flamework-flickrapp)

* [flamework-api](https://github.com/straup/flamework-api)

* [flamework-invitecodes](https://github.com/straup/flamework-invitecodes)

* [flamework-tools](https://github.com/straup/flamework-tools)