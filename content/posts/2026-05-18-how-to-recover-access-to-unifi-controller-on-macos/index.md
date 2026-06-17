+++
draft = false
date = 2026-05-18T17:32:39+02:00
title = "How to recover access to a UniFi Controller on macOS"
description = ""
slug = "how-to-recover-access-to-unifi-controller-on-macos"
author = "Bernhard Frick"
tags = ["ubiquiti", "unifi", "controller", "mongodb", "shadow"]
categories = ["networking", "databases"]
externalLink = ""
series = []
+++

Recently, I got my hands on a used 8-port UniFi Switch with PoE
([US-8-60W](https://techspecs.ui.com/unifi/switching/us-8-60w)) that needed to
be reset before it can be used. Easy - just plug it in, hold down the reset
button, start the controller software that is running on my MacBook, "Click to
Adopt" and we're off to the races.

Somewhere in this process, my diligence of storing access credentials in a
password manager has failed me and I found myself locked out of my own network
configuration. The "Reset Password" option did not work because the macOS
installation of the UniFi Controller was missing SMTP configuration, and the top
suggestion on the internet was to start over with a new controller and
reconfigure everything from scratch.

Luckily, after a few minutes of investigating, I found that the "UniFI Network
Server" version 9.0.114 runs an internal MongoDB 3.6 database, that contains the
password.

**Step 1: Connect to the MongoDB database server**

The MongoDB server is running on the non-standard port 27117. Connecting to it
with `mongosh` version 2.8.3 (`brew install mongosh`) fails because `mongosh`
does not support wire version 6 anymore:

```shell
$ mongosh --port 27117
Current Mongosh Log ID:	6a0b35e3d75f5365de02a7c0
Connecting to:		mongodb://127.0.0.1:27117/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.8.3
MongoServerSelectionError: Server at 127.0.0.1:27117 reports maximum wire version 6, but this version of the Node.js Driver requires at least 8 (MongoDB 4.2)
```

Instead, I had to install the older `mongo` client that is compatible with
MongoDB 3.6:

```shell
brew tap mongodb/brew
brew install mongodb/brew/mongodb-community-shell
```

After that, I could connect to the MongoDB server without authentication:
```shell
mongo --port 27117
```

**Step 2: Find the password hash in the database**

The database `ace` contains a collection called `admin`, that contains the
UniFi Controller admin users and their password hashes:

```shell
db.getSiblingDB("ace").admin.find({}, {"name": true, "email": true, "x_shadow": true}).pretty()
```

Output:
```json
{
    "_id" : ObjectId("6573a73eedd4b85523f2a5db"),
    "x_shadow" : "$6$n6ZXA6Sh$KJnAYClb9UX3Gwz43xkQa5p0bR8IjBJ.Uhjcz7JpUhUnmlsaqrT.otSf1zAFYEZ2TA8lqKBpgZ/79SPAln81K.",
    "name" : "admin",
    "email" : "admin@example.com"
}
```

**Step 3: Generate a new password**

The UniFi controller stores passwords in the same format as the `/etc/shadow`
file in Linux:

```
$[algorithm]$[salt]$[hash]
```

In this case, `6` indicates an SHA-512 hash.

To generate a new hash, I used `mkpasswd`:

```shell
mkpasswd --method=SHA-512
```

**Step 4: Update the admin user password in MongoDB**

I updated the admin user in MongoDB to set the new password hash:

```js
db.admin.updateOne({"_id" : ObjectId("6573a73eedd4b85523f2a5db")}, { $set: { "x_shadow" : "$6$Kt563EOE4oSMrTLS$XMJKS8zH2y36dzgRrlW9NrTVvCc0lfw6IEZSSWh9tGZKIalVRbQpTFTXXgXdm4c2cxSOf2r1xrUzFG1N5oeAF." } })
```

**Step 5: Log in with the new password**

Finally, I was able to log in to the UniFi Controller again, and adopting the
new PoE switch went swimmingly from there. I think I know what my next evening
project will be - configuring a more stable deployment that also includes
automated configuration backups :) 

## Sources

* https://community.ui.com/questions/Lost-UniFi-Controller-Login-Password/61513566-c3b9-4128-9fa6-84034c083311
* https://linuxize.com/post/etc-shadow-file/
* https://unix.stackexchange.com/questions/81240/manually-generate-password-for-etc-shadow
