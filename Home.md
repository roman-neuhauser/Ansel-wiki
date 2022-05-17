Welcome to the R&Darktable wiki!

Things that got removed from upstream darktable:

* the [slideshow view](https://docs.darktable.org/usermanual/3.8/en/slideshow/): darktable is simply too slow for this view to be responsive enough. UNUSABLE
* the [tethering view](https://docs.darktable.org/usermanual/3.8/en/tethering/): it relies on the lib GPhoto2 which is unmaintained, it's super brittle and buggy. UNUSABLE.
* the easter egg video game. USELESS.
* the histogram overlay control for [exposure and black level](https://docs.darktable.org/usermanual/3.8/en/module-reference/utility-modules/shared/scopes/#exposure-adjustment) : it's too easy to inadvertently mess up exposure when cycling through scopes (especially with a Wacom) and it adds unnecessary motion events and redraws. DANGEROUS. LOSS OF CPU CYCLES.
* the new filtering option : 6300 lines of bloat and counting to fix a problem nobody had with something that adds bugs and trouble. REGRESSION.
* the [stars rating](https://docs.darktable.org/usermanual/3.8/en/lighttable/digital-asset-management/star-color/#star-ratings) toolbar in lighttable : it's redundant with overlays and requires to travel too far with the cursor to set it. Use overlays with cursor or key shortcuts to set stars. USELESS.
* the [timeline](https://docs.darktable.org/usermanual/3.8/en/module-reference/utility-modules/lighttable/timeline/) : too complicated for the little it does. LOSS OF CPU CYCLES.
* the main window borders with collapsing arrows: they waste space and tend to reappear when you don't want them because their state is view-dependent. LOSS OF DISPLAY REAL-ESTATE.