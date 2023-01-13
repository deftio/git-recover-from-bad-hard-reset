# Recovering from git reset --hard (and its friends)

So I screwed up and accidentally hit a 

```sh
git reset --hard
```

when I should have been paying attention more and lost all the files in my initial commit.  

~~ admonishment etc ~~

But here is how I unwound that (not easy but doable if git hasn't garbage collected your commits yet):

## First we need to find lost commits:

```sh
git fsck --lost-found
```

which shows stuff like this

```text
notice: HEAD points to an unborn branch (359d54df943b1769eb18093f41c3c3c2f4438f54)
notice: No default references
missing tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904
dangling tree 359d54df943b1769eb18093f41c3c3c2f4438f54
```

Normally you may be able to restore some of these lost files with

```sh
git checkout 4b825dc642cb6eb9a060e54bf8d69288fbee4904
```

but at for me returned this:

```
fatal: Cannot switch branch to a non-commit '4b825dc642cb6eb9a060e54bf8d69288fbee4904'
```

or 

```sh
git reflog
```
but that returned this:

```
fatal: your current branch '359d54df943b1769eb18093f41c3c3c2f4438f54' does not have any commits yet
```

and of course this doesn't work either:

```sh
git checkout 34053495some-long-hash2340293034334
```

and this didn't work either for the same reason... 

```sh
git cherry-pick git checkout 359d54df943b1769eb18093f41c3c3c2f4438f54
```


so you may have dangling trees or commits which can't be restored via the normal means...

Alas all *may* not be lost (yet)..


## The amazing power of git cat-file...

SO back to the output of 
```sh
git fsck --lost-found
```

It turns out we can take that dangling tree and run this:

```sh
git cat-file -p 359d54df943b1769eb18093f41c3c3c2f4438f54
```

which shows the contents of that dangling tree (that git checkout, etc can't crack open)  giving this:

```
18093f41c3c3c2f4438f54
100644 blob 936d605963eb73d0021c58965c5e7a3c3752ce03    README.md
040000 tree bfa5598456b567cebde5a6a5917b7ab4a5d24f5d    config
040000 tree db92d1e1ffb392836e19d2923cec45ca4b228126    mosquitto
040000 tree 3eed7d61bde8c636fdac0ea2072638c787882f64    node_modules
100644 blob 12277d965af7e05539bc1e361939d1a1fa680995    package-lock.json
100644 blob 0ca4858f81be9a782a65cb31f6dd1d4be1cbf52f    package.json
040000 tree 768502931d4ae5a641717441531bc1dcf2067a85    src
```

and then again to descend in to the src directory...

```sh
git cat-file -p 768502931d4ae5a641717441531bc1dcf2067a85
```

which showed all the files in the /src directory like this:

```
717441531bc1dcf2067a85
100644 blob 6139a8a8cfee274645c44940761238d7fcc4cb61    mqtt-cli.js
100644 blob 40b47fee9de8ff291dd380db7e2960bce1f9e266    mqtt-hive.js
100644 blob 05b90da77155109af8a3686792fcabacb95bcf2d    mqtt-local-pub.js
100644 blob 607bd7f8d3a7d2302f0bc73b04675f1c6230cb8b    mqtt-local-sub.js
100644 blob 2a58b0ec63c0203ecea66b2e1b129cdd3789734d    mqtt_publisher.py
100644 blob be2afb5ea21d77f61a1e43c43811275d8c06cf37    mqtt_subscribe.py
```

which finally could be restored like this..



```sh
git cat-file -p 607bd7f8d3a7d2302f0bc73b04675f1c6230cb8b >  mqtt-local-sub.js
```
yup just like that you can rescue some lost files out of the file system abyss.

There are some scripts around to automate this but atleast you can navigate through the hashes of the old commit history and have some luck.

I can't say this will undo bad git resets but atleaset there is a chance...!

Good luck!

## Some excellent references:

* [Yashints](https://yashints.dev/about) : 
[How I recovered a day's worth of work with git](https://yashints.dev/blog/2020/04/05/git-logs)

* [Stack Overflow: recover-dangling-blobs-in-git](https://stackoverflow.com/questions/9560184/recover-dangling-blobs-in-git)

* [Rescue your ever lost commits](https://github.com/pendashteh/git-recover-index)


