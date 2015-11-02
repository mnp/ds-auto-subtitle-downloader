## What and Why

This small bit of duct tape allows a Synology Diskstation to
automatically download subtitle **.srt** files after the Download
Station app runs. The Video Player app can web search for subtitles
but it's an extra clicky step and it's not very reliable.

## How

The Downloader app has a post-download script hook, so we will use
that to call our **update-subs** script to search a given subfolder
(mine is set to /volume1/video; yours may differ), and for every video
file it discovers, if there's not already an .srt file in the same
directory, it will fetch one. It uses
https://github.com/arshad/subdb-cli, which is much better at finding
subtitles, and a mangled piece of
https://gist.github.com/simov/4132717 to recurse.  We also use a cron
job to re-install ourselves after the DS overwrites the hook we put
in, every time it upgrades the Download Station app.

## Installation

1. On the DS control panel, turn on the SSH server if you haven't already.
2. On the DS app store panel, install the **node.js** and **Download Station** apps.
3. Copy the **update-subs** and **patch-download-settings** to your DS to the root folder.  How you do this is probably specific to your setup.
4. Open a console and SSH into your DS.
5. Install a dependency for Node:
```
YourDS> npm install -g subdb-cli
```
6. Install the project files where they belong, then patch our hooks into the
   Download Station app settings file.
```
YourDS> chmod a+x update-subs patch-download-settings
YourDS> mv patch-download-settings update-subs /usr/local/bin
YourDS> /usr/local/bin/patch-download-settings
```
7. We need to add a cronjob to do this again after the DS updates.  If
   you look at /etc/crontab, youll notice there's line like this,

```
56	1	*	*	1,2,3,4,6	root	/usr/syno/bin/synopkg chkupgradepkg
```

which means every Mon, Tue, Wed, and Sat at 01:56 AM, check for upgrade. We will re-run our settings patcher an hour later. You can do the edit from the command line.

```
YourDS> cp /etc/crontab /etc/crontab.bak
YourDS> echo '56	2	*	*	1,2,3,4,6	root	/usr/local/bin/patch-download-settings' >> /etc/crontab
```

## Bugs

Yes, included. PR's welcome.

