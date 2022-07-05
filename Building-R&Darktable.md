R&Darktable diverges from upstream darktable at commit [1bdf797445639f536f4f6effbb44ed40c678a810](https://github.com/aurelienpierre/R-Darktable/commit/1bdf797445639f536f4f6effbb44ed40c678a810) (7th May 2022) which is the first of the reverting commits of darktable's lighttable filters. 

Therefore, if you checkout on tags or commits published before this date (like `git checkout release-3.8.1` or `git checkout release-3.9.0`) on this repository, you will get upstream darktable.

# Building R&Darktable synced with upstream

TL;DR: `git checkout master` or `git pull https://github.com/aurelienpierre/R-Darktable.git master`

__R&Darktable exists for now only in the [master branch](https://github.com/aurelienpierre/R-Darktable/tree/master)__. This `master` branch is kept in sync with upstream darktable (at least twice a month, but not real-time) for everything related to pixel code, such that you can transparently edit images with one or the other, and switch to the other later, without breaking edits. 

Keep in mind that the difference between R&Darktable and darktable is only in the GUI code and in some presets/preferences. Pixel modules are the same. In any case, the master branch is still an unstable branch (as in "the branch where new stuff gets merged for general testing, pending a release") both on upstream darktable and here.

# Building R&Darktable with bleeding edge stuff

TL;DR: `git checkout dev` or `git pull https://github.com/aurelienpierre/R-Darktable.git dev`

The experimental new modules and other things that __will break your edits__ and __not be compatible with upstream darktable `master`__ are merged in the [dev](https://github.com/aurelienpierre/R-Darktable/tree/dev) branch since 2018. This is the branch I personally use for my editing because I understand what's going on in there, but I don't recommend anybody else to use it on a daily basis or for serious work. 

The `dev` branch is basically R&Darktable's `master` in which I merge the topic branches where I develop new stuff that gets submitted as PR (pull requests). To use it, you need to `git checkout dev` and backup your database and XMP sidecars before doing anything.

# Actually installing R&Darktable

R&Darktable uses the same scripts and building stack as upstream darktable, so everything works the same but you need to be aware that it will get installed in place of darktable and as darktable, so it uses the same pathes and default config/database directories. The CLI commands are all the same and R&Darktable will be registered as darktable as far as your OS is concerned.

I'm not sure at this point if it's worth it or even desirable to have it installed somewhere else and using different config pathes and CLI commands.

# Installing R&Dt alongside upstream darktable

It is possible to install both alongside but you need to manually change the installation directory, like such : 

```shell
./build.sh --prefix /opt/rdarktable --build-type Release --install --sudo
```

Then start the app with:

```shell
/opt/rdarktable/bin/darktable
```

while the upstream darktable variant would be started simply with `darktable` or `/opt/darktable/bin/darktable`.

If you want to use another configuration directory, for safety, you can R&Dt with:

```shell
/opt/rdarktable/bin/darktable --configdir "~/.config/darktable-test"
```

The R&Dt logo has been changed in-app such that you can easily identify in which app you are. The system icon may still be the upstream darktable one. This will gradually be improved.

It is not recommended to install R&Dt nor dt in virtual machines since color management can't be ensured. Most display color profiles make use of the [VCGT](https://argyllcms.freelists.narkive.com/uTZlLmgY/vcgt-meaning-with-regard-to-the-icc-profile#post2) and it is not clear if VM can make proper use of it, which ultimately depends on how they support and access GPU drivers. 