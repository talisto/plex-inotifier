# Plex iNotifier

Plex is an amazing piece of software.  One issue with Plex Media Server, though, is that it doesn't auto-update the library when there are changes on connected network shares.  This can be annoying if you run your PMS on a different machine (e.g. Nvidia Shield, Mac Mini, etc) than where your media library is stored (e.g. NAS), as you either have to trigger an update on PMS manually when adding/moving/renaming content or wait for the library update interval to kick in. 

So, I wrote a script to automate that (borrowed heavily from [here](https://codesourcery.wordpress.com/2012/11/29/more-on-the-synology-nas-automatically-indexing-new-files/)).

The script works by tying into inotify on the NAS, which is a kernel subsystem that notices changes to the filesystem and reports those changes to applications.  The script uses inotify to monitor the media directories on the NAS for changes, then connects to the remote Plex Server's web API to find the appropriate media section to refresh.  If it finds a matching section, it uses the web API to send an update command to that section.


## Installation

1. Make sure you have "Python3" installed.

2. You'll need to install the "pynotify" Python module.  The easiest way is to install the Python EasyInstall utility; Shell into your server, and run:
 `wget https://bootstrap.pypa.io/ez_setup.py -O - | python3` then run: `easy_install pyinotify`

3. Save the `plex-inotify.py` script somewhere on your NAS/fileserver, e.g. `/usr/local/bin` 

4. Edit the `plex_server_host` variable near the top of your script to match the IP address of your Plex Server.  If you have local DNS resolution, you can use a hostname instead.

5. Edit the `path_maps` variable to map the local paths of the media shares on your fileserver to their corresponding library names in your Plex Media Server.

6. You should change the `daemonize` variable to `False` for testing purposes until you're sure that everything is working properly.

7. Try running the script with `python3 plex-inotify.py`, and if all goes well, it will load up without errors :)

If you set `daemonize` to `True`, then the script will fork itself into a background task when you run it.  It will stay running even if you log out of the shell.

## Troubleshooting

* If you see a bunch of errors that say something like `Errno=No space left on device (ENOSPC)`, then your inotify watcher limit is too low.  Run `sysctl -n -w fs.inotify.max_user_watches=16384` and then try again.  Keep raising the number until the errors go away.

* If you see an error that says `Errno=No such file or directory (ENOENT)`, then you didn't configure your `paths_maps` properly.  Make sure each entry in the list is a local path to your media on the NAS and then the corresponding library/section name on your PMS.

* If you're getting an error that says `urllib.error.HTTPError: HTTP Error 401: Unauthorized`, then you need to set the `plex_account_token` variable.  Follow [this link](https://support.plex.tv/hc/en-us/articles/204059436-Finding-your-account-token-X-Plex-Token) for instructions on how to get your account token.  Make sure that when you're setting the variable, you wrap the token in quotes, like this: `plex_account_token = 'A2ekcFXjzPqmefBpv8da'`

Let me know if you find this useful!