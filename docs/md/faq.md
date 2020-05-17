<p>
    <a name="installation"></a>
</p>

### Lychee is not working
If Lychee is not working properly, try to open `https://lychee.example.com/Diagnostics`. This script will display all errors it can find.

### What do I need to run Lychee on my server?
To run Lychee, everything you need is a web server with PHP 7.3 or later and an SQL database.

Lychee uses a `.htaccess` file to configure the server and allow for some advanced options, like uploading from other sources via the API. If you're running an Apache server, you need to enable `.htaccess` write permissions.

You can configure Apache globally in `/etc/apache2/apache2.conf` or per-directory in `/etc/apache2/sites-available/your-website.conf` (exact path and file names are distribution-dependent). Find the `<Directory>` directive, change `AllowOverride None` to `AllowOverride All` and save.

```
<Directory /var/www/your-site>
  AllowOverride All
</Directory>
```

### I can't upload photos
If you experience problems uploading large photos, you might want to change the PHP parameters in `.htaccess` (if you are using the PHP Apache module) or in `.user.ini` (if you are using PHP >= 7.3 with CGI or FastCGI).

> If you modify the `.user.ini` file, you may want to run `git update-index --assume-unchanged .user.ini` afterwards.

If possible, change these settings directly in your `php.ini`. We recommend to increase the values of the following properties:

```
max_execution_time = 200
post_max_size = 100M
upload_max_size = 100M
upload_max_filesize = 20M
memory_limit = 256M
```

### What does _Upstream sent too big header_ error message mean?
This error may be seen from your browser's console if you're trying to debug something with Lychee. If using `nginx`, try adding the following to Lychee's config and reload nginx's service:
```
fastcgi_buffers 16 16k;
fastcgi_buffer_size 32k;
```

### Which browsers are supported?
Lychee should work with any modern, standards-compliant web browser (note that having JavaScript enabled is a hard dependency). Lychee does take advantage of several HTML5 features, such as the `<video>` tag, responsive image sizing using `srcset`, or link data prefetching. In particular, Lychee supports the latest versions of Google Chrome, Apple's Safari, Mozilla Firefox, Opera, and Microsoft Internet Explorer.

If you experience any issues with Lychee and wish to report it, make sure to specify the browser you're using and the version of it.

### Which file formats are supported?
Lychee supports major image formats, and since version 3.2.1 some video formats as well. Specifically, `*.jpg`, `*.jpeg`, `*.png`, `*.gif`, `*.ogv`, `*.mp4`, `*.webm`, and `*.mov` are accepted.

If you're uploading video files, make sure to increase your upload limits in `php.ini`.  See the [Install](https://github.com/LycheeOrg/Lychee-Laravel/wiki/Install#72-php) section for more information.

### What is new?
Take a look at the [Changelog](Changelog) to see what's new.

### How can I install Lychee without SSH access ?

1. Download the latest release
2. Extract and upload the folder via ftp
3. access the website.
If you are at the wrong address you will be told to go to the `public` folder.
Once open, you will be redirected to the install procedure. Completed you will be able to create an admin account and enjoy Lychee.

### How do I upgrade from Lychee v3 to Lychee v4?

The process without docker supports the migration pretty well:

Assuming the following tree:
```
/var/
  |- www/
         |- html/
              |- Lychee/
              |- <you are here in html>
```

You can do:
```sh
git clone https://github.com/LycheeOrg/Lychee-Laravel
cd Lychee-Laravel
# apply composer to install dependencies
composer install --no-dev
# move the pictures
mv ../Lychee/uploads/big/* public/uploads/big/
mv ../Lychee/uploads/medium/* public/uploads/medium/
mv ../Lychee/uploads/small/* public/uploads/small/
mv ../Lychee/uploads/thumb/* public/uploads/thumb/
```
From here there are two ways:

1. open `public/` with your browser and follow the steps.
2. apply the following steps in command line:
  ```
  # copy .env
  cp .env.example .env
  # edit dot env
  vim .env
  # don't forget [:] [q] to quit ;)
  php artisan migrate
  # generate a secure key to encrypt the cookies
  php artisan key:generate
  ```

And finally, make apache point to the `public/` directory and enjoy.


### How can I easily contact the LycheeOrg organization ?
There is a gitter associated with the project, feel free to join us there: https://gitter.im/LycheeOrg/Lobby

### How can I set thumbnails for my albums?
Thumbnails are selected automatically from the photos inside the album (and any subalbums) based on the photo sorting order specified in the [Settings](Settings#sorting). _Precedence is given to starred photos_. In practical terms, if only one photo inside an album is starred, that photo is guaranteed to be the top thumbnail.

### Is it possible to create folders inside another folder? If so, how to do that?

Either press `n` (as **N**ew) or use the add menu.

![Screenshot (33)](https://user-images.githubusercontent.com/627094/80621138-2c5fac00-8a47-11ea-849e-050f740d3b7d.png)


### How can I back up my installation?
To back up your Lychee installation you need to perform the following steps:

1. Create a copy of at least the following parts of the Lychee directory tree (e.g., `/var/www/html/Lychee`):
```
.env
public/dist/user.css
public/uploads/
```
2. Dump the Lychee database to a file. E.g., if you are using MySQL, run:
```
mysqldump -u user -ppassword --databases lychee_database > lychee_backup.sql
```
Replace `user`, `password`, and `lychee_database` by the values of `DB_USERNAME`, `DB_PASSWORD`, and `DB_DATABASE` from the `.env` file in the Lychee folder.

### How can I migrate my installation to a new host?

To move your Lychee installation, you need to perform the following steps:

1. Copy the whole Lychee folder (e.g., `/var/www/html/Lychee`) to the new host.
2. Dump the Lychee database to a file. E.g., if you are using MySQL, run:
```
mysqldump -u user -ppassword --databases lychee_database > lychee_backup.sql
```
Replace `user`, `password`, and `lychee_database` by the values of `DB_USERNAME`, `DB_PASSWORD`, and `DB_DATABASE` from the `.env` file in the Lychee folder.

3. Restore the database on the new host, e.g., for MySQL:
```
mysql -u user -ppassword < lychee_backup.sql
```

### Can I use my existing folder structure?
Not at this time. Lychee currently uses its own folder structure and database. Please upload or import all your photos to use them.

### Can I upload videos?
Yes, but you will need to change this property to a bigger value:
```
upload_max_filesize = 20M
```

### Can I host Lychee-Laravel with a subpath with Nginx? Like `https://example.dev/lychee/`

Yes, here is a configuration to help you:

```nginx
location ^~ /lychee {
    alias /var/www/lychee/public;
    index index.php;
    try_files $uri $uri/ @lychee;
    location ~ \.php$ {
        if (!-e $request_filename) {
            rewrite ^/lychee/?(.*)$ /lychee/index.php?/$1 last;
            break;
        }
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $request_filename;
    }
}
location @lychee {
    rewrite /lychee/(.*)$ /lychee/index.php?/$1 last;
}

```

### Why don't my videos have thumbnails?
You will need ffmpeg installed on your server, and to have installed php-ffmpeg using composer as detailed in the [Installation Guide](https://github.com/LycheeOrg/Lychee-laravel/wiki/Installation).

If you are still having problems, check your Lychee log. If you are still getting a `Could not create thumbnail for video because FFMPEG is not available.` error, you may need to specify the location of your ffmpeg and ffprobe binaries. In https://github.com/LycheeOrg/Lychee-Laravel/blob/master/app/ModelFunctions/PhotoFunctions.php#L94 replace
```
$ffmpeg = FFMpeg\FFMpeg::create();
```
with
```
$ffmpeg = FFMpeg\FFMpeg::create(array(
        'ffmpeg.binaries'  => '/usr/bin/ffmpeg',
        'ffprobe.binaries' => '/usr/bin/ffprobe',
));
```
using your correct binary locations. If unsure, you can try running `which ffmpeg` on the server.

### Is it possible to create multiple users?

Yes. Just go to the `Users` menu.

### Does Lychee use ImageMagick?
Yes. Lychee uses ImageMagick when available.

### How to change the title of the site?

Go to the advanced Settings menu and change the value of `site_title`.

### How to reset username and password?

via ssh, use the command `php artisan lychee:reset_admin`

### How to hide smart albums?

Add the following custom CSS to your `user.css` or via the settings menu:
```css
[data-id="0"] { display:none; }
[data-id="s"] { display:none; }
[data-id="f"] { display:none; }
[data-id="r"] { display:none; }
```

### How to disable the 'zoom' animation while browsing pictures?

Add the following custom CSS to your `user.css` or via the settings menu:
```css
#imageview #image {
  transition: none !important;
  animation-name: none !important;
  animation-duration: 0 !important;
}
```

### Composer can't create a cache directory

* When running Composer, you may notice the following warning:
```
Cannot create cache directory /home/$USER/.composer/cache/files/, or directory is not writable. Proceeding without cache
```
* You can specify Composer's cache directory with the environment variable `COMPOSER_CACHE_DIR=`. For Lychee, the cache is not necessary, and you can both disable it and hide the warning by specifying the location of the cache as `/dev/null` ([information](https://github.com/composer/composer/commit/fd6455218e304e9b484bebb0efcdb67bb52d051d)):
```
COMPOSER_CACHE_DIR='/dev/null' composer update --working-dir='/var/www/Lychee'
```

### Do we really need writable `app/`?

From [#311](../issues/311)

Short answer: Lychee-Laravel will work without a writable `app/` folder.

However, if you want to be able to update your Lychee installation with a click of a button in the web browser (which is supported with Lychee-Laravel, though it's disabled by default), your whole Lychee installation tree (and not just `app/`) must be http-writable.

As far as I know, the minimum set of directories that need to be http-writable is as follows:
```
storage/framework/sessions
storage/framework/views
storage/logs
public/uploads
public/uploads/small
public/uploads/big
public/uploads/thumb
public/uploads/medium
public/uploads/import
public/dist
```

### How is the upload folder protected?

From [#304](../issues/304)

Short version: It's not protected

Long answer: [#295](../pull/295) added some protection through symlinking, so that the URLs used are temporary.

Right now, the protection is basically through the use of difficult to guess names (it's an MD5 checksum of the system time at the time of upload). [#295](../pull/295) not only made those names temporary (this needs to be enabled in the Settings, BTW) but it also provided optional support for hiding the full-size version (this is only effective with symlinking as without it the URL can be derived from that of intermediate-size images).

@ildyria recenlty posted the following link on how a more effective protection could be implemented: https://bedigit.com/blog/laravel-5-how-to-access-image-uploaded-in-storage-within-view/. He didn't go down that route himself due to performance concerns but I agree that if somebody contributed a clean implementation as an option, we'd probably accept it.

### My login is timing out after two hours, how can this be changed?

You can edit your `.env` and modify the `SESSION_LIFETIME=120` part (in minutes).

### How can I clear the Logs? 

To remove the `Notice` and `Warnings`, simply click on the button at the top of the Log page.
However if the page is too heavy to load, you can either manually empty the `Logs` table or use artisan:

```
cd Lychee
php artisan lychee:logs clean
```

### Why can't I see the *check for update* button in the GUI?

- Make sure you are using a `git` installation and not a downloaded release
- Make sure you are on the `master` branch
- Make sure your web user has read and write access to `.git`
- Check that `exec` is available.
- Check that 'allow_online_git_pull' setting is set to '1'


### Is it possible to do the update directly from the GUI? How?

You go to `Diagnostics` -> `Check for Updates`  
Once done you need to update the `Diagnostic` page (click on it again in the left menu) 
You will see a `Apply Update` button on the top. Click on it and done.
If it breaks (error 500) you can still go back to command line and do your `git pull`, migrate etc...

There are some securities that you need to disable via the advanced settings menu:

- you need to allow update if your .env specify `production`:  
  `force_migration_in_production` = `1`
- you need to allow composer :  
  `apply_composer_update` = `1` (optional)

The second one is really optional if updates don't need the composer (like 90% of the time) then it can just stay at `0`.

### Can I migrate from a 64-bit system to a 32-bit system?

Yes, but it's not trivial or recommended. After copying the database:
1. Download [this](https://github.com/LycheeOrg/Lychee-Laravel/raw/54d00878949906c2efd4f6ddd9e79669637c58fb/database/migrations/2019_04_07_193345_fix_32bit.php) file to your `database/migrations/` folder.
2. Run  the SQL command `delete from migrations where migration='2019_04_07_193345_fix_32bit';` to make sure it will run.
3. Run `php artisan migrate`. This should run a one-off migration that was originally added to allow 32-bit systems to migrate from Lychee v3.

This will only work on top-level albums. Subalbums will require [manual intervention](https://github.com/LycheeOrg/Lychee-Laravel/issues/406#issuecomment-571378073).

### How can I see the correct client IP address when running Lychee behind Cloudflare?]

Please see https://github.com/monicahq/laravel-cloudflare. The Lychee file that needs changing can be found [here](https://github.com/LycheeOrg/Lychee-Laravel/blob/master/app/Http/Middleware/TrustProxies.php).

### Can I set up Lychee to watch a folder for new images and automatically add them to albums?

Lychee can import photos from the command line using `php artisan lychee:sync /path/to/import`. Folders in this path will be converted to albums, and subfolders as subalbums using `php artisan lychee:sync /path/to/import --album_id="album ID"`.



This command can be scheduled using `cron` or a systemd timer.
```sh
man crontab
man systemd.timer
```


### Special right-click menu doesn't appear for new users?

For example:  
![](https://user-images.githubusercontent.com/14360110/74829349-22b41c80-5311-11ea-96e9-adb648a2c3fe.png)

No this is normal. This user does not have the ownership of that Album, so the right click is not available. You can see _sharing_ as a _read_ permission.

> Also, the divider h1 shows the text "Admin" when logged in with an ordinary user. I guess this should be "Albums"?

Actually no, this is because the user does not have any albums (yet). The `h1` divider is to show who is the owner of those albums. See bellow.
![2020-02-19_3840x1080_12:29:19](https://user-images.githubusercontent.com/627094/74830496-92c3a200-5313-11ea-9065-60cb8090c7ac.png)
And yes the right-click menu is available on the _PhD Defenses_ part but not in the _Admin_ parts.
Also because this user has upload right, he can see the _Unsorted, Public, Starred, Recent_ smart albums. 

### I can't access the users under settings server error or api not found Lightspeed
If you receive a server error or api not found error under lightspeed web server try going to cPanel > Mod Security and turning the feature off.

### How can I stop `artisan migrate` and `composer` from running after every `git pull`?
We've set composer to install git `pre-commit` and `post-merge` hooks by default.
- The pre-commit is to help developers by checking and fixing and code style problems before they hit our testing. If you aren't committing and changes, this will not be run.
- The post-merge ensures that your dependencies and database version are kept current. This can be disabled by creating a file `.NO_AUTO_COMPOSER_MIGRATE` in your Lychee root and deleting (if necessary) `.git/hooks/post-merge`. If you disable this script, you will need to run `composer` and `artisan migrate` manually. Alternatively, the Update UI can be set to handle this if you run your updates there.