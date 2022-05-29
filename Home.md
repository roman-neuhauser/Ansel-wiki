Welcome to the R&Darktable wiki!

The philosophy of R&Darktable is : 

> If it's half working, then it's fully gone.

# What was removed from mainstream darktable

## Features

* the [tethering view](https://docs.darktable.org/usermanual/3.8/en/tethering/): it relies on the lib GPhoto2 which is super brittle and buggy. The tethering view is merely a GUI over Gphoto2, much like [Entangle](https://entangle-photo.org/). The problem is GPhoto2 is known to break something different at each new release, so it's simply not worth the trouble to maintain a GUI over a Schrödinger cat. UNUSABLE. Use Entangle if you need to.
* the easter egg video game. USELESS.
* the histogram overlay control for [exposure and black level](https://docs.darktable.org/usermanual/3.8/en/module-reference/utility-modules/shared/scopes/#exposure-adjustment) : it's too easy to inadvertently mess up exposure when cycling through scopes (especially with a Wacom) and it adds unnecessary motion events and redraws. In cases of multiple instances of exposure module, it is automatically linked to the controls of the last instance, which is generally wrong. DANGEROUS & LOSS OF CPU CYCLES.
* the new filtering option : 6300 lines of bloat and counting to fix a problem nobody had with something that adds bugs and trouble. REGRESSION.
* the [stars rating](https://docs.darktable.org/usermanual/3.8/en/lighttable/digital-asset-management/star-color/#star-ratings) toolbar in lighttable : it's redundant with overlays and requires to travel too far with the cursor to set it. Use overlays with cursor or key shortcuts to set stars. USELESS.
* the [timeline](https://docs.darktable.org/usermanual/3.8/en/module-reference/utility-modules/lighttable/timeline/) : too complicated for the little it does. LOSS OF CPU CYCLES.
* the main window borders with collapsing arrows: they waste space and tend to reappear when you don't want them because their state is view-dependent. LOSS OF DISPLAY REAL-ESTATE.
* the [second darkroom window](https://docs.darktable.org/usermanual/3.8/en/darkroom/darkroom-view-layout/) : darktable is slow enough with 2 pipelines (thumbnail & preview), a third one is not realistic. This should have been the main preview redirected to another window but the code has been hastily hacked by copy-pasting, it's terrible. Also there is no traceability on which display profile will be used. LOSS OF CPU CYCLES.
* the block and timeout overlay mode in lighttable : terrible code for something never used. USELESS.

## Preferences

* The option to display a mask button in module's header is removed and the button is always shown,
* The option to display the scroll bars in lighttable and darkroom is removed, scrollbars will always appear in lighttable and will never appear in darkroom,
* The option to favour speed at the expense of quality is removed (process downscaled thumbnails), quality is favoured.
* The option to expand modules on "enable" event and collapse on "disable" event is removed, collapsed state is independent from enabled state.
* The option to scroll to the expanded module is removed, expanded modules will always trigger a scroll to be fully visible in viewport.
* The option to open only one module at a time is removed. Modules will never be auto-collapsed when a new module is expanded.

# Modules deprecated

All the remaining Lab modules that I have warned against for years are deprecated, as well as the not-so-good RGB ones. Namely:

* bloom,
* raw chromatic aberrations,
* contrast, lightness, saturation,
* colorize,
* color mapping,
* high-pass,
* low light,
* low-pass,
* raw denoise,
* shadows and highlights,
* sharpen
* soften,
* split-toning,
* velvia

One of the most common questions, since the big scene-referred change, was "what modules should I avoid". Now, you don't need to ask.

# What was changed from mainstream darktable

## Behaviours

* In lighttable, the "mouse over" event now does not select images for writing and possibly harmful operation (writing metadata, copying history stack, deleting/moving files, applying styles, rating, labelling, tagging etc.). These events trigger only safe read-only events (displaying metadata).
* The "copy all" history stack now copies and pastes white balance too.

## Widgets

* The export module is moved to the left sidebar in darkroom and in lighttable, for consistency,
* "collections" module renamed "collect",
* "history stack" module renamed "apply development",
* "geotagging" module renamed "edit geo tags",
* "selected image[s]" module renamed "act on selection", and its "images" tab renamed "files",
* "metadata editor" module renamed "edit metadata",
* "image information" module renamed "display metadata" and moved at the bottom of the left sidebar, to be always visible if expanded (Head-Up Display style),
* "recently used collections" module renamed "recent collections",
* "styles" module renamed "apply styles",
* "tagging" module renamed "edit text tags",
* the rating module (stars) at the bottom of the lighttable is removed since rating can already be made on overlays and using shortcuts.

The purpose of these changes is to have a "file" left sidebar in lighttable (import, export, collect) and an "edit" right sidebar (apply styles, developments, tags, etc.), with names that call to action in order to present the workflow steps.

---

# Total of lines of code removed

Excluding comments and blanks, using `cloc $(git ls-files)`:

* C code and C headers : 14074
* CSS : 564