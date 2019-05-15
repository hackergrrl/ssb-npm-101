# ssb-npm 101

status: draft

## what is ssb-npm?

it's like npm, except the registry of packages (package metadata + tarballs)
lives entirely on secure scuttlebutt

this means there is no central authority controlling packages!

## prerequisites

1. have npm 5 or newer (relies on `package-lock.json`; won't work with earlier
   npm versions); can do `npm install -g npm@latest`
2. have [scuttlebot](https://github.com/ssbc/scuttlebot) installed, with `sbot
   server` running successfully on your machine

## workaround: big blobs

ssb-npm-registry depends on `sodium-native` which is larger than `sbot`'s
default maximum blob size. To get around this, you can either

1. edit your ssb config (usually `~/.ssb/config`) to include `{"blobs":{"max":10000000}}`, or
2. run `sbot` as `sbot server --blobs.max 10000000`

to be able to get the blob for `sodium-native`.

## ssb-npm-registry

ssb-npm-registry is an npm registry, not unlike the one running on npmjs.org,
except this registry uses secure scuttlebutt to find packages that were
published to it, and installs them from there!

ssb-npm-registry itself is actually published to ssb, but without a locally
running registry we'll have to install it manually. once we do that, we can use
the `ssb-npm` command just like the regular `npm` command to install packages.

first, we'll pull down the blob for the latest version of `ssb-npm-registry`.
you can find out what the latest blob is by searching `npm-packages` packages
with `sbot`:

```
$ sbot messagesByType npm-packages | grep -C 1 ssb-npm-registry
```

you'll see something like this near the bottom of the output:

```
  "name": "npm:ssb-npm-registry:1.7.0:latest",
  "link": "&2afFvk14JEObC047kYmBLioDgMfHe2Eg5/gndSjPQ1Q=.sha256",
```

you can now use `sbot` to WANT and then GET that blob, which is the npm tarball
of the package:

```
$ sbot blobs.want '&2afFvk14JEObC047kYmBLioDgMfHe2Eg5/gndSjPQ1Q=.sha256'
$ sbot blobs.get '&2afFvk14JEObC047kYmBLioDgMfHe2Eg5/gndSjPQ1Q=.sha256' >
ssb-npm-registry.tar.gz

$ tar xvzf ssb-npm-registry.tar.gz

$ mv package ~/.ssb/node_modules/ssb-npm-registry
```

you'll need to add the entry to your `"plugins"` section of your `~/.ssb/config`
file:

```
  "ssb-npm-registry": true
```

Assuming you had an empty ~/.ssb/config before, and added the `blobs: max` part from before, this would now look like this:

```
   {
     "blobs": {"max": 10000000},
     "plugins": {"ssb-npm-registry": true}
   }
```

now you can restart `sbot server`.

## ssb-npm

now we can install the `ssb-npm` command. what's nice is that since the registry
is installed locally, we can actually use the vanilla `npm` command to do so:

```
$ npm install ssb-npm --global --registry=http://localhost:8043/
```

woo, now you can use `ssb-npm install ...` to install packages by name, just
like the regular `npm` command!

## publishing packages

let's publish one of your own npm packages to ssb-npm:

in that module's root directory, run

```
$ ssb-npm publish
```

you'll see your module published. :tada:

what about your dependencies though? unless all of your module's dependencies
are already on ssb-npm (this is unlikely depending on what you're working on;
the ssb registry is still very young), you'll need to publish those as well. you
can publish all of your module's dependencies that aren't already on ssb using
`ssb-npm-migrate`:

```
$ ssb-npm-migrate
```

and then republish:

```
$ ssb-npm publish
```

and you're done. now anyone that is connected to your friend graph can run
`ssb-npm install pkg` to install your package, all without touching any oldweb
services like npm!

## license

CC0

