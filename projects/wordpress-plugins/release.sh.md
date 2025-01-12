## Working with release.sh

Many of our plugins use a shell script, `release.sh`, to handle pushing new versions to `wordpress.org`'s plugins SVN repository. The canonical copy of this file [is in this repository](./release.sh).

This file provides setup, usage, and configuration instructions for `release.sh` in your WordPress plugins.

### The first release

1. Review the files in the `OMIT_LIST` variable in `release.sh`. Make changes as necessary.
2. Update the `SVN_REPO` variable with your plugin's `plugins.svn.wordpress.org` SVN repository.
3. `chmod +x release.sh`.
4. Add `release/` to the project's `.gitignore`.
5. Commit all changes.
6. Tag your new release using `git tag`.
7. `git push` your commits and `git push --tags` your tags to the plugin's repository.
8. Run `release.sh`.
  - You may need to verify the receiving server's key fingerprint.
  - If it asks you to enter the password for your computer's username, press the `[enter]` key to get the username prompt, then enter your `wordpress.org` username and password.
  
### Using a new device with an existing plugin repository
1. Review the files in the `OMIT_LIST` variable in `release.sh`. Make changes as necessary.
2. Update the `SVN_REPO` variable with your plugin's `plugins.svn.wordpress.org` SVN repository.
3. `chmod +x release.sh`.
4. `mkdir release`.
5. `svn co https://plugins.svn.wordpress.org/plugin-slug/ release` to make the `release` directory. 
6. `cd ..` to get back to the root of the plugin.
7. Run `release.sh`.

### Later releases

1. Commit all changes.
2. Make sure that you have bumped the version number in:
	- `readme.txt`
	- `README.md`
	- the plugin's primary `.php` file
3. Make sure that you have also updated:
	- `Tested up to:` WordPress version number in `readme.txt`
	- `Requires at least:` WordPress version number in `readme.txt`
	- `Requires PHP:` minimum PHP version number in `readme.txt`
	- `Changelog` section in `readme.txt`
2. Tag your new release using `git tag`.
3. `git push` your commits and `git push --tags` your tags to the plugin's repository.
4. Run `release.sh`.

### Common issues


#### no remote configured
```
$ ./release.sh
fatal: No remote configured to list refs from.
fatal: No remote configured to list refs from.
Bad release state for git repo!
Make sure you've checked out a tag or the master branch before releasing.
```

Make sure that one of your remotes is named `origin`.

#### "forbidden" when committing

```
Committing release/svn/tags/1.7 (slow) ...
Authentication realm: <https://plugins.svn.wordpress.org:443> Use your WordPress.org login
Password for 'user': 
svn: E195023: Commit failed (details follow):
svn: E195023: Changing directory '/Users/local_username/sites/example/wp-content/plugins/PLUGIN-NAME/release/svn/tags/1.7' is forbidden by the server
svn: E175013: Access to '/!svn/txr/RANDOM-CHARACTERS/PLUGIN-NAME/tags/1.7' forbidden
```

A possible cause of this is that your WordPress.org account does not have commit access. Check to make sure that your WordPress user has the "committer" role on wordpress.org for this plugin, as [described in the Plugin Handbook](https://developer.wordpress.org/plugins/wordpress-org/special-user-roles-capabilities/).

#### certificate error

```
Error validating server certificate for 'https://plugins.svn.wordpress.org:443':
 - The certificate is not issued by a trusted authority. Use the
   fingerprint to validate the certificate manually!
Certificate information:
 - Hostname: *.svn.wordpress.org
 - Valid: from Fri, 12 Jun 2015 17:44:41 GMT until Sun, 15 Jul 2018 19:04:26 GMT
 - Issuer: http://certs.godaddy.com/repository/, GoDaddy.com, Inc., Scottsdale, Arizona, US
 - Fingerprint: 5c:f0:21:33:a0:f1:f6:37:ac:06:87:c8:62:03:08:d0:32:50:6f:77
(R)eject, accept (t)emporarily or accept (p)ermanently? p
```

According to https://make.wordpress.org/core/handbook/tutorials/installing-wordpress-locally/from-svn/#1-2-using-command-line the accepted solution is "Type `p` to accept it permanently."

#### `.git/` directories contained within the plugin's release .zip file

Check your plugin's `release.sh` to make sure that `./\*\*/.\*` is contained within the `OMIT_LIST` variable.

https://github.com/INN/docs/blob/1e53bd3a6793ea8c7197670316e49c9ff6da8661/projects/wordpress-plugins/release.sh#L19

This line is a [bash glob](http://www.tldp.org/LDP/abs/html/globbingref.html) of the form `./**/.*`, which says: in the current directory, in any child directory of any depth, match files or directories the name of which starts with a `.`. This should catch `./vendor/composer-dependency/.git/`, `./node_modules/dependency/node_modules/other-dependency/node_modules/yes-really/.git/`, and also files like `./node_modules/zlib-browserify/.travis.yml`, `./js/vendor/jquery/src/.eslintrc.json`, and really anything found by the command `find . -type f -name '.*'`.

### Generating zip files for GitHub

GitHub's default release `.zip` files do not contain files installed by package management systems through commands like `npm install`, `bower install`, and `composer install`.
`release/wp-release.zip` is generated by `release.sh` for every release on the master branch, and contains everyhting that is put in the wordpress.org plugin repository.

If you don't have a recent `wp-release.zip`, generate one in the following manner:

1. Run `release.sh --dry_run`
2. When it asks you if you really want to deploy, type `y`: it will not actually deploy, because of the `--dry_run` flag.
3. Check for the existence of `release/wp-release.zip`.

Once you have `release/wp-release.zip`, upload `release/wp-release.zip` to GitHub as [a binary asset for the release](https://github.com/blog/1547-release-your-software). This can only be done for [tagged releases](https://help.github.com/articles/creating-releases/).

### Updating the `assets/` directory of `svn/`

`release.sh` only affects the `tags/` and `trunk/` directories. To update the assets contained in the `assets/` branch, such as [plugin assets](https://developer.wordpress.org/plugins/wordpress-org/plugin-assets/) for the WordPress.org listing, perform the following steps:

1. Add files to `svn/assets/` by whatever means you want
2. From `svn/`, run `svn add assets/*`
3. Check that the files are marked for addition with `svn status`
4. svn commit -m "Updating assets"
  - You may need to verify the receiving server's key fingerprint.
  - If it asks you to enter the password for your computer's username, press the `[enter]` key to get the username prompt, then enter your `wordpress.org` username and password.

WordPress.org's [How to use Subversion](https://developer.wordpress.org/plugins/wordpress-org/how-to-use-subversion/) article may be of interest, but it's focused on releasing releases and not updating assets.

### Plugins using `release.sh`

This list may be incomplete.

- The original script: https://github.com/publicmediaplatform/pmp-wordpress/blob/master/release.sh
- The current script: https://gist.github.com/rnagle/40d84cbd5fef86e3de7781fc31b46d94
- https://github.com/INN/analytic-bridge
- https://github.com/INN/DoubleClick-for-WordPress
- https://github.com/INN/link-roundups
- https://github.com/INN/super-cool-ad-inserter-plugin
- https://github.com/INN/pym-shortcode

Significant changes to `release.sh` should be copied between plugins, and merged into this repository.
