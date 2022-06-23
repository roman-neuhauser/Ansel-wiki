R&Darktable diverges from upstream darktable at commit [1bdf797445639f536f4f6effbb44ed40c678a810](https://github.com/aurelienpierre/R-Darktable/commit/1bdf797445639f536f4f6effbb44ed40c678a810) (7th May 2022) which starts the reverts of darktable's lighttable filters. 

Therefore, if you checkout on tags or commits published before this date (like `git checkout release-3.8.1` or `git checkout release-3.9.0`) on this repository, you will get upstream darktable.

# Building R&Darktable synced with upstream

__R&Darktable exists for now only in the [master branch](https://github.com/aurelienpierre/R-Darktable/tree/master)__. This `master` branch is kept in sync with upstream darktable (at least twice a month, but not real-time) for everything related to pixel code, such that you can transparently edit images with one or the other, and switch to the other later, without breaking edits. 

Keep in mind that the difference between R&Darktable and darktable is only in the GUI code and in some presets/preferences, pixel modules are the same, but the master branch is still an unstable branch (as in "the branch where new stuff gets merged for general testing, pending a release").

So you need to `git checkout master` before building.

# Building R&Darktable with bleeding edge stuff

The experimental new modules and other things that __will break your edits__ and __not be compatible with upstream darktable `master`__ are merged in the [dev](https://github.com/aurelienpierre/R-Darktable/tree/dev) branch since 2018. This is the branch I personally use for my editing because I understand what's going on in there, but I don't recommend anybody else to use it on a daily basis or for serious work. 

The `dev` branch is basically R&Darktable's `master` in which I merge the topic branches where I develop new stuff that gets submitted as PR (pull requests). To use it, you need to `git checkout dev` and backup your database and XMP sidecars before doing anything.
