---
author: jet
categories:
- Homelab
- Linux
- Media
- Networking
- Server
- Tech
date: "2021-06-27T12:39:52Z"
guid: https://jetvillegas.com/?p=4
id: 4
tags:
- configuration
- Linux
- Migration
- Plex
- Server
title: Plex Server Migration
url: /2021/06/27/plex-server-migration/
---

In this post, I describe how to migrate media from one Plex server to a new server installation. The [official Plex documentation](https://support.plex.tv/articles/201370363-move-an-install-to-another-system/) on how to do this leaves out several critical details that I’ll cover here. If you’ve somehow lost the server configuration options from your new install, I’ll show you how to fix that below.

<figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/no_server_options-1024x897.png)</figure>## Prepare the Source Server

I’m moving from one Linux installation (Ubuntu 18.04.5 LTS) to another (Ubuntu 20.04.2 LTS.) I’m also moving from a bare metal server to a KVM virtual machine but these instructions should work (with some platform-specific adjustments) for your system.

The first thing to do is to sign out of the server you’re migrating from. We’ll call this the *source* server.

<figure class="wp-block-image size-large">![](http://jetvillegas.com/wp-content/uploads/sites/5/2021/06/settings_button.png)<figcaption>Click on the Settings button in Plex to get to the server configuration options.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/sign_out_server-1024x923.png)<figcaption>Click Settings &gt; General &gt; Sign Out on the source server.</figcaption></figure>Next, we’ll stop the Plex Media Server service on the source server. On Linux, you do this with:

`$ sudo service plexmediaserver stop`

This will differ on Windows or Mac, but same idea.

Confirm that the service is stopped on the source server. Again, this will differ on Windows or Mac, but same idea:

`$ sudo service plexmediaserver status`

Move the media files to the new location. You can move the files any way you like, but I personally use rsync for its fault tolerance and how it correctly preserves file permissions.

`$ sudo rsync -zarvh --progress you@source_server:/path/to/your/source/plex/media/ /path/to/your/plex/destination/media`

Note the trailing / on the source directory will copy the contents of the media folder into a new folder named media on the destination. Depending on how much media you have, this can take a while.

- - - - - -

## Prepare the Destination Server

Install Plex Media Server on the *destination* server. [Download the latest version](https://www.plex.tv/media-server-downloads/) from Plex and follow the installation steps for your server platform.

For Plex to work, it must have access over TCP port 32400. If you run a firewall, you’ll need to allow that:

`$ sudo ufw allow 32400`

Start the Plex Media Server service on the destination server:

`$ sudo service plexmediaserver start`

Check that it’s running OK:

`$ sudo service plexmediaserver status`

<figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/server_status-1024x296.png)<figcaption>If all’s well, you should see it running on the destination server</figcaption></figure>Note where Plex installs itself and its configuration files. On Linux, it’s installed here:

Server Config: `/var/lib/plexmediaserver/Library/Application Support/Plex Media Server`

Server Install: `/usr/lib/plexmediaserver`

**Note:** The Plex Media Server runs as the user `plex` by default. The `plex` user must have read and execute permissions to your media directories and files. Verify that there is a `plex` user on your system. On Linux, you can check the `passwd` file for a user named `plex`.

`$ less /etc/passwd`

To complete the migration you must log into the destination server on `localhost`. It’s important that you do this with a web browser by typing `http://localhost:32400/web` on the address bar. If you skip this step, you will have missing server options and no access to your media libraries. You have to complete the installation on `localhost`.

If you installed Plex Media Server on a headless server with no local web access, you have to tunnel into the destination server from your local machine. This tricks Plex into thinking you’re logging in from localhost.

We’ll tunnel over port `8888`. On Linux you do this over SSH. If you have a Firewall running, you’ll need to allow port `8888`:

`$ sudo ufw allow 8888`

Next, SSH into the destination server:

`$ ssh you@ip.address.of.server -L 8888:localhost:32400`

On your local computer, type `http://localhost:8888/web` on your browser’s address bar to continue.

<figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_first_sign_in-1024x933.png)<figcaption>Sign into the destination server on `localhost` using your Plex email and password.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_friendly_name-1012x1024.jpg)<figcaption>Give your server a friendly name.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_got_it-928x1024.png)<figcaption>After naming your destination server, Plex will offer up options for re-adding your media libraries.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_organize_media-1024x958.png)<figcaption>Click Add Library to start adding back your media files on the destination server.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_organize_media_add_library-1024x745.png)<figcaption>Pick a Library type depending on the files you’re adding back.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_organize_media_add_movies-1024x738.png)<figcaption>Browse for the folder on your destination server that contains the Library’s files.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_organize_media_add_movies_2-1024x797.png)<figcaption>Libraries you add get added to the list that Plex will index and process.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_organize_media_finished-1024x969.jpg)<figcaption>Click Done after you’ve finished adding Libraries.</figcaption></figure><figure class="wp-block-image size-large">![](https://jetvillegas.com/wp-content/uploads/sites/5/2021/06/localhost_8888_organize_media_revealed-717x1024.png)<figcaption>You’ll now see all your media in the Libraries listed below the default ones. </figcaption></figure>- - - - - -

## Cleaning Up

If you had to tunnel into your server’s firewall, remove port `8888` after you’re done

`$ sudo ufw delete allow 8888/tcp`

I personally like running Plex over https. I also dislike using port 32400 so I set it up with nginx as a reverse proxy. That’s outside the scope of this post, but [here’s a nice configuration file](https://github.com/toomuchio/plex-nginx-reverseproxy) that I found on github for that purpose.