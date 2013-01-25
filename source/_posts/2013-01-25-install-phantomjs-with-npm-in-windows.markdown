---
layout: post
title: "Install PhantomJS with NPM in Windows"
date: 2013-01-25 14:38
comments: true
categories: 
---

When we install phantomjs(1.8.0) with npm in Windows by `npm install phantomjs`, the process may fail and throw out an error like this:

    Error with http request { connection: 'close',
      location: 'http://124.29.1.238/download/20544659/21242686/3/zip/156/114/1359054422428_370/phantomjs-1.8.0-windows.zip'
     }

The error is due to a defective network handling in phantomjs' `install.js` script, since it requests a zip file in `phantomjs.googlecode.com` but doesn't handle the http redirection correctely.

What's worse, the npm won't allow us to install a package if one of its dependencies fail to install, so if we can't install phantomjs, we can't install other packages depend on it independently.

So here come the following method, but this is not a general way to handle similiar problems. At least it works in this case.

## Try First
Let's try to install `svg2png`(which depends on phantomjs) by

`npm install -g svg2png`

As expected it will fail, but now npm has downloaded the phantomjs package into its cache dir. The default cache dir is

`%USERPROFILE%\appdata\Roaming\npm-cache`

And we can change it by

`npm config set cache=xxx`

Now there must be a `phantomjs` dir, and it may contain these files:

    phantomjs\1.8.0-1\package\xxx
    phantomjs\1.8.0-1\package\install.js
    phantomjs\1.8.0-1\package.tgz
    phantomjs\.cache.json

The `package.tgz` is the archive that npm will extract it every time when it needs to install phantomjs. And the `package\xxx` are the contents of the archive. So we need to modify the code, repack a new package.tgz, replace the original one, and install again.

## Download Zip
There is a `downloadUrl` variable in the `package\install.js` file, which points to a archive file, we need to download it manually and save it in some convenient place.

In my case, this url is 

`http://phantomjs.googlecode.com/files/phantomjs-1.8.0-windows.zip`

And I save it in

`d:\node\phantomjs-1.8.0-windows.zip`

## Modify Package
Modify `phantomjs\1.8.0-1\package\install.js` in the cache dir. Skip the network process and directly extract the archive file. The patch:

{% gist 4632657 installjs.patch %}

Be careful with the slash's direction in the code

    var zipPath = path.normalize('d:/node/phantomjs-1.8.0-windows.zip')

Remove original `package.tgz`, then create a new one by `tar cxzf package.tgz package`.

## Install Again
Run again

`npm install -g svg2png`

If everything is fine, this time it will succeed.
