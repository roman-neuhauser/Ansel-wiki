Welcome to the R&Darktable wiki!

The philosophy of R&Darktable is : 

> If it's half working, then it's fully gone.

# What is R&Darktable ?

Rational & Darktable is a darktable fork with 22229 lines of bad code removed. _(Updated 19th July 2022)_

Only the general-purpose and robust features have been kept, in order to focus on efficient image-processing and reduce the overhead:

1. **It runs slightly faster than upstream darktable**: although not the ultimate goal, removing GUI events leads to less CPU load
2. **It provides a minimal set of tools and simplified GUI** and doesn't overwhelm users with loads of geeky options which don't make them more productive. 
3. **It solves the long-standing problem of re-inventing basic GUI interaction** in ways that are inconsistent with every operating system and desktop environment out there. 
4. **It keeps pixel code and pipeline compatibility with upstream darktable**: you have the choice of your UI, you keep the compatibility of your edits.

The reasons for having forked are detailed here:

* [Why a fork ?](https://github.com/aurelienpierre/R-Darktable/wiki/Why-a-fork-%3F),
* [Open source and professional photography : lies and wishes](https://eng.aurelienpierre.com/2022/06/open-source-and-professional-photography-lies-and-wishes/),
* [Design by committee will not save FLOSS](https://eng.aurelienpierre.com/2022/06/design-by-committee-will-not-save-floss/).

# What is **not** R&Darktable ?

1. **It does not aim at making darktable faster** : the pixel pipeline code is exactly the same, and darktable already has a pipeline as fast as it will ever be under the architecture and design chosen in 2009. Improving performance is the goal of [vkdt](https://github.com/hanatos/vkdt), which will not be ready for production for at least a couple of years, and is a full rewrite with a different pipeline. But, of course, removing GUI bloat is going to speed some things up.
2. **It is not a beginner-friendly photo-editing app** : the goal is to simplify the overhead, meaning everything not directly related to outputting processed images. That is software configuration/options, GUI disposition, file management, labels, etc. Color science is still what it is, and boy, it's no easy.
3. **It is not a community project** : community is great, except when it grows too much too fast and you don't have any kind of management to keep a unified line of development and a consistent design. More people means more help until you reach a team of 4-5, above this you need to allocate resources to management, so more people means more work. This is exactly what has happened to darktable: lots of hype since v3.0 leading to 2 releases/year with almost 2000 commits in each, so everything gets merged too soon and more bugs are introduced than fixed at each release, all that to keep momentum.
4. **It is not a hot project** : I consider darktable's development (and therefore R&Darktable's one) roughly 90% finished, meaning it's close to feature-complete. We are reaching the limits of its initial design & architecture regarding how much we can still extend it without actively harming performance and stability. The goal right now is to polish existing features, fix the worst usability pain points and the low-hanging fruits, then keep it stable pending [vkdt](https://github.com/hanatos/vkdt)'s availability. But the darktable's team does not want to hear it and wants to keep adding as much as possible in it.

See also : [The future of darktable](https://github.com/aurelienpierre/R-Darktable/wiki/The-future-of-darktable).

# Support

So far, R&Darktable is pretty close to upstream darktable (see the details of the changes below), so there is no reason to get excited. There are no packages yet and it's a work in progress. If you need help with something, please use the Matrix channels:

* In English : https://matrix.to/#/#r&darktable-en:matrix.org
* En Français : https://matrix.to/#/#r&darktable-fr:matrix.org

R&Darktable builds the same and with the same dependency stack as upstream darktable, see [how to build](https://github.com/aurelienpierre/R-Darktable/wiki/Building-R&Darktable).

# Roadmap

(Updated June 22nd 2022).

The immediate goal for now is:

1. to get rid of the last useless preferences,
2. finish backporting non-dynamic culling view features to the dynamic one,
3. publish a 4.0 version compatible with upstream darktable 4.0 pipeline (modules),
4. get an OBS repository up and going, maybe with a Flatpak thingy (though Flatpak sucks).

The mid-term goal is:

1. to rewrite the module groups with something simpler with fixed modules layout, closer to what was in darktable 3.0,
2. remove some options from the great MIDI turducken (at least the triple click thing and the shortcuts for Gtk notebook tabs… the shortcut window is currently a nightmare to configure due to way too many possible shortcuts).

The long-term goal is anyway to transition to [vkdt](https://github.com/hanatos/vkdt/), which simplifies the pipeline (GPU-only, full-resolution rendering – no heuristics), breaks free from the performance bottleneck that is Gtk by using [Imgui](https://github.com/ocornut/imgui) (low-bloat and running straight on GPU), and keeping GUI code __really__ separated from pixel code. Because darktable's core has deep design flaws that have no foreseeable solution under the constraint of preserving compatibility with older edits, so the only solution is a rewrite.

That is to say, I will pick here the low-hanging fruits, fix what can be easily fixed, to build a reliable tool usable while we wait for vkdt to be mature enough for production.

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
* the extended overlays mode for thumbnails: each of the EXIF metadata display triggered its own SQL request per-thumbnail. LOSS OF CPU CYCLES.
* the non-dynamic culling mode and the zoomable lighttable view : they are not really used and need a lot of spaghetti code to be handled. DANGEROUS and USELESS.
* the dark and icon themes : dark themes trigger all sorts of unwanted visual illusions that make color assessment impossible. DANGEROUS.
* the ability to assign keyboard shortcuts to Gtk notebook tabs, as well as switching tabs by mouse scroll, has been removed. USELESS and INCONSISTENT WITH UI PRINCIPLES.
* the VIM-style shortcuts : feature never documented that still listens to all keystrokes and applies its checks. 

Overall, 5 to 6 SQL requests per thumbnail per "mouse over" event are now spared, due to the features removed.

## Preferences

* The option to display a mask button in module's header is removed and the button is always shown,
* The option to display the scroll bars in lighttable and darkroom is removed, scrollbars will always appear in lighttable and will never appear in darkroom,
* The option to favour speed at the expense of quality is removed (process downscaled thumbnails), quality is favoured.
* The option to expand modules on "enable" event and collapse on "disable" event is removed, collapsed state is independent from enabled state (I was the author of that stupidity and even I grew weary of this).
* The option to scroll to the expanded module is removed, expanded modules will always trigger a scroll to be fully visible in viewport.
* The option to open only one module at a time is removed. Modules will never be auto-collapsed when a new module is expanded.

# Modules deprecated

All the remaining Lab modules that I have warned against for years are deprecated, namely:

* bloom, *(replaced by diffuse or sharpen, preset blooming)*
* raw chromatic aberrations, *(replaced by RGB chromatic aberrations)*
* contrast, lightness, saturation, *(replaced by filmic and color balance)*
* colorize, *(replaced by color balance)*
* color contrast, *(replaced by color balance)*
* color correction, *(replaced by color balance)*
* color mapping, *(replaced by color mapping features in exposure and color calibration)*
* color reconstruction, *(replaced by filmic color reconstruction, "high quality reconstruction" method)*
* high-pass, *(replaced by diffuse or sharpen, various deblurring presets)*
* levels, *(replaced by filmic, white and black relative exposures settings)*
* low light, *(replaced by color balance)*
* low-pass, *(replaced by blurs)*
* monochrome, *(replaced by color calibration, "grey" tab, and various film-emulation presets)*
* raw denoise, *(replaced by denoise profiled)*
* shadows and highlights, *(replaced by tone equalizer and local contrast)*
* sharpen, *(replaced by diffuse or sharpen, various deblurring presets)*
* soften, *(replaced by diffuse or sharpen, blooming preset)*
* split-toning, *(replaced by color balance)*
* tone curve, *(replaced by tone equalizer, filmic and color balance)*
* velvia. *(replaced by color balance)

The display-referred RGB modules have been deprecated too:

* base curve, *(replaced by filmic)*
* RGB levels, *(replaced by filmic, white and black relative exposures settings)*
* RGB curves, *(replaced by tone equalizer, filmic and color balance)*

One of the most common questions, since the big scene-referred change, was "what modules should I avoid". Now, you don't need to ask. Despite what some people say, "deprecated" means the modules can't be found in UI for new edits, but they are still very much in the program and ready to serve for old edits that use them. So deprecating is just GUI cleaning.

The *output color profile* module has been removed from the darkroom GUI. It is **still** applied in the pipeline, so only the GUI widget is hidden, but since the output profile is set either from the display options (for the thumbnails and previews) or in the export module (for file exports), the profile defined in the darkroom module was overwritten anyway, and the duplicated setting confused some users. In the export module, the former "same as image" profile option has been replaced by "same as original", which will reuse the embedded color profile if any or fall back to sRGB otherwise (noticeably for raw files).

# What was changed from mainstream darktable

## Behaviours

* In lighttable, the "mouse over" event now does not select images for writing and possibly harmful operation (writing metadata, copying history stack, deleting/moving files, applying styles, rating, labelling, tagging etc.). These events trigger only safe read-only events  in the metadata display module.
* Similarly, the modules like tagging, geotagging, etc. that reacted to the "mouse over" event (by updating their content) now only react to selection (click or key stroke). This saves many SQL requests to read metadata when moving the mouse in lighttable and improves responsiveness.
* The "copy all" history stack now copies and pastes white balance too.
* The CSS animations have been removed. Animations are basically videos that need to be computed at 30 FPS while Gtk is only single-threaded on CPU,
* The mouse scroll will scroll the content of the container (sidebars) and will not be captured by sliders, unless sliders got clicked over first (or used through keyboard shortcuts). This restores the intuitive and common behaviour of a mouse scroll and protects from unwanted setting changes. See #11.

## Widgets

* Modules names, buttons and section labels have their initial letter capitalized programmatically as grammar and typography rules suggests for better visual cues and for consistency with the rest of the world. This does not break translations but may trigger weird behaviours with non-european languages (untested).
* The "export" module is moved to the left sidebar in lighttable, for consistency with import, collect, etc.
* The "export" and "tagging" modules are removed from darkroom, since they are DAM features.
* "import" module renamed "open & import", its buttons renamed "open from disk" and "import from camera",
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

## Presets

* In the scene-referred workflow, the base presets have been changed like follow:
  * in exposure module, the exposure boost has been changed from +0.5 to +0.7 EV,
  * in filmic module, the contrast is changed to 1, latitude is set to 0.01% (which practically disables it — doesn't matter for v6), and the interpolation spline is set to back to "hard" (fourth order polynomial with $C^2$ connectivity condition).