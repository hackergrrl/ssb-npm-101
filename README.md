# ssb-npm 101

status: draft

## what is ssb-npm?

it's like npm, except the registry of packages (package metadata + tarballs)
lives entirely on secure scuttlebutt

this means there is no central authority controlling packages! it also means you
can use npm in offline environments.

## prerequisites

1. have npm 5 or newer (relies on `package-lock.json`; won't work with earlier
   npm versions); can do `npm install -g npm@latest`
2. have [scuttlebot](https://github.com/ssbc/scuttlebot) installed, with `sbot
   server` running successfully on your machine

## two approaches

From here, there are two ways of using ssb-npm. Both are equivalent; it's just a
preferred interface / setup choice.

### 1. ssb-npm command

This is, in the author's opinion, the easiest way to use ssb-npm. It means
running the `ssb-npm` command when you want to use ssb as your registry, and
running regular `npm` when you want to use npmjs.org as your registry. For
example

```
$ ssb-npm install
```

to install all dependencies inside of a module directory. This works because the
`ssb-npm` package bundles the `ssb-npm-registry` inside of it.

The `ssb-npm` package is published to `ssb-npm` itself! So we can use the `sbot`
command to query your scuttlebutt node to find the latest version:

```
$ sbot messagesByType npm-packages | grep -A 1 npm:ssb-npm: | grep -o '&.*sha256' | tail -n 1
```

Try running the commands incrementally to get a feel for what's going on here
(e.g. first `sbot messagesByType npm-packages` then add the `| grep -A 1
npm:ssb-npm:` part, and so forth. What this is doing is querying your local ssb
database for all published npm packages (the `npm-packages` message type), and
then narrowing it down to the `ssb-npm` package, and finally narrowing that down
to the latest version.

The result will be something like
`&SItHiY5GLYh+LTcxqgIolBpAWzFjybG1bk3rQ3JZikw=.sha256`. This is a blob
identifier, which is a block of binary data. In this case, since it's a
published npm module, it's actually a `.tar.gz` tarball. We can use `sbot` to
download it:

```
$ sbot blobs.want '&SItHiY5GLYh+LTcxqgIolBpAWzFjybG1bk3rQ3JZikw=.sha256'
$ sbot blobs.get '&SItHiY5GLYh+LTcxqgIolBpAWzFjybG1bk3rQ3JZikw=.sha256' > ssb-npm.tar.gz
```

This tells our sbot node we're interested in the blob, and then retrieve its
contents. From here we can tell our vanilla `npm` program to install the package
globally right from the tarball:

```
```


### 2. ssb-npm-registry plugin

This approach is running the ssb-npm-registry plugin on your local scuttlebot
node. Interaction with ssb-npm means using the normal `npm` command but with the
`--registry` switch. So an `npm install` would look instead like

```
$ npm install --registry=http://localhost:8043/
```

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
```

now untar the tarball and put it in your `~/.ssb/node_modules` folder so `sbot`
can find it:

```
mv package ~/.ssb/node_modules
```

you'll need to add the entry to your `"plugins"` section of your `~/.ssb/config`
file:

```
  "ssb-npm-registry": true
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

