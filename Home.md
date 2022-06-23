Welcome to the R&Darktable wiki!

The philosophy of R&Darktable is : 

> If it's half working, then it's fully gone.

# What is R&Darktable ?

Rational & Darktable is a darktable fork with 21525 lines of bad code removed. _(Updated 23rd June 2022)_

It runs slightly faster than upstream darktable and doesn't overwhelm users with loads of geeky options which don't make them more productive. It aims at doing the bare minimum really well instead of trying to (over)do everything at once in a sub-optimal way. It solves the long-standing problem of re-inventing basic GUI interaction in ways that are inconsistent with every operating system out there. 

# Why a fork ?

Even I don't have the slightest idea of what many preferences in darktable do. The problem is all those preferences/options trigger special behaviours that may interfere with each other and produce edge cases (heuristics) difficult to predict, debug, refactor and simplify. They make the code hard to read and therefore hard to maintain. But "cool new stuff" is favoured over maintainability and ease of use. 

That wouldn't be so bad if so many changes since 2020 hadn't been just options for the sake of it. Triple click support in shortcuts declaration ? Timeline view ? Modules groups with presets ? Lighttable overlays options ? Nobody needed them, which would be fine if most of them were not broken in subtle ways. 

Let's run a quick computation : there are 7 flavours of thumbnails overlays × 5 different lighttable views × 3 base color themes = 105 combinations of options to style individually in CSS themes… Guess how many are currently broken in upstream darktable ? 3 overlays × 3 views was more than enough, but no… We had to make it impossible and annoying for everyone involved.

This habit, now grounded in darktable's DNA, to keep piling up useless options makes darktable more difficult to learn, understand, use, debug and maintain. This is not progress, this is not design (aka the process of solving problems through the simplest path), this is just geeky kindergarten to keep middle-aged middle-class white men busy with code on saturdays. It's typical Linux bullshit where people enjoy the feeling of power they get from an overwhelming and cryptic UI more than the result of the work they achieve through it. Breaking news: software is a tool, a tool is meant to do a job, only the result matters and only the idiots fall in love with their hammer.

Many GUI interactions in darktable don't comply with what's deemed standard in 2022, and not even with Gnome or KDE defaults, because apparently we need to outsmart everyone by reinventing even the most basic things like keyboard navigation and mouse hover events. Sorry, but the 2020's is not the right time to pretend to reinvent GUI interactions, and the result is anyway not as clever or usable as the devs think.

Actually, the harsh truth is very few of these many options make you more productive. But they surely lose your time since you need to read the doc or (re)configure shortcuts to be able to perform basic tasks like bulk files selection or filtering. And since your preferences get nuked at pretty much each update now, the time you spent tuning and tweaking all that cannot even be seen as a long-term investment: it's just a dry loss.

Finally, the abuse of Gtk events and poorly-coded custom widgets makes the soft sluggish and laggy since too many things happen "real-time" whenever you move the cursor in the window, simply because too many things capture mouse events for disputable tasks, often duplicated. Gtk is a disgrace (single-threaded on CPU only and overall inefficient), let's keep its use the the strict minimum, shall we ?

Basically, git pulling darktable now is an anxiety-triggering task meant to answer a daunting question: "what did they break this time ?". I have had enough of that shit. Adding solutions in search for a problem is not ok when it degrades basic usability. Defending solutions with "me likey" is childish and stupid, the quality of a tool is to be assessed by the number of steps and the time needed to achieve a specified task, and what people (devs and users) like is irrelevant.

Let's do less better. That is, remove the bloat, stop trying to account for every anecdotal sub-optimal workflow, and make actual management choices and decisions to build a consistent set of tools for a bunch of real, defined, use cases.

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
* the non-dynamic culling mode and the zoomable lighttable view : they are not really used and need a lot of spaghetti code to be handled. DANGEROUS and USULESS.
* the dark and icon themes : dark themes trigger all sorts of unwanted visual illusions that make color assessment impossible. DANGEROUS.

Overall, 5 to 6 SQL requests per thumbnail per "mouse over" event are now spared, due to the features removed.

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

* In lighttable, the "mouse over" event now does not select images for writing and possibly harmful operation (writing metadata, copying history stack, deleting/moving files, applying styles, rating, labelling, tagging etc.). These events trigger only safe read-only events  in the metadata display module.
* Similarly, the modules like tagging, geotagging, etc. that reacted to the "mouse over" event (by updating their content) now only react to selection (click or key stroke). This saves many SQL requests to read metadata when moving the mouse in lighttable and improves responsiveness.
* The "copy all" history stack now copies and pastes white balance too.
* The CSS animations have been removed. Animations are basically videos that need to be computed at 30 FPS while Gtk is only single-threaded on CPU.

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

## Presets

* In the scene-referred workflow, the base presets have been changed like follow:
  * in exposure module, the exposure boost has been changed from +0.5 to +0.7 EV,
  * in filmic module, the contrast is changed to 1, latitude is set to 0.01% (which practically disables it — doesn't matter for v6), and the interpolation spline is set to back to "hard" (fourth order polynomial with C2 connectivity condition).

# Who the f** is that bloke anyway ?

I'm mostly a portrait photographer with a double training in classical music (piano) and hardware engineering (mechanics and metrology). I taught music for a while and I currently teach post-processing with darktable. 

I have been an user of darktable since its 0.9 version, somewhere circa 2011. At that time, I didn't even know how to program. I joined darktable's development in 2018 at first to solve the color issues that arose when editing large dynamic range pictures, which led to filmic and later filmic RGB modules, then to the whole scene-referred workflow. Before 2018, I did mostly B&W photography because I couldn't get the colors I wanted. I gradually extended the scene-referred pipeline with the ASC-CDL mode in color balance, then color balance RGB, tone equalizer, color calibration (and its built-in color checker profiling tool), and negadoctor for film digitizing. I'm happy to report than I now get the colors I wanted and that darktable lets you build your own virtual film emulsion.

I like minimalistic design, so I build generic tools that can be used for a variety of uses (including unplanned and uncharted uses) rather than specific tools that essentially duplicate the same pixel code under different interfaces for the sake of guiding (but restraining) users. 

I also like physics and clean math, with a deep interest for research. Building on the physical accuracy of the scene-referred pipeline, I developed the blurs module which simulates lens and motion blur with physically-accurate models. I extended the Fourier [heat equation](https://en.wikipedia.org/wiki/Heat_equation) to the 4th order in a [wavelets framework](https://en.wikipedia.org/wiki/Spline_wavelet#Cubic_B-spline) and to anisotropic diffusion, at first to produce watercolour-like smudges, and then figured it could be extended to deblurring and dehazing by simply subtracting instead of adding the diffusion term… which gave birth to the diffuse or sharpen module in darktable 3.8. Though I still have mixed feelings about enabling the sharpness addicts to overcook every picture in ways that don't improve its artistic quality.

Reusing that 4th order diffusion model in wavelets space, I invented the joint guided laplacian + RGB ratios diffusion method to reconstruct clipped highlights in 3D, added in darktable 4.0. This one accidentally qualifies as a supervised machine learning algorithm for gradient-based optimization. Fancy words to say that we reconstruct damaged parts of the image by filling them with synthetic content that extrapolates the gradients from the valid parts in a smooth way.

In early 2022, I developed a new [uniform color space](https://eng.aurelienpierre.com/2022/02/color-saturation-control-for-the-21th-century/) specifically for the needs of the color balance RGB, that allows to manipulate saturation for artistic purposes with account for the [Helmholtz-Kohlrausch effect](https://en.wikipedia.org/wiki/Helmholtz%E2%80%93Kohlrausch_effect).

Developing for darktable is my full-time job, paid by the users who think it's worth it, and it's no secret that I couldn't have done all that in less than 4 years otherwise. It wasn't my goal to become opensource dev, but since nobody else would fix all that, there is not much of an alternative. There are worse jobs though than designing artistic tools for visual expression in a methodic and rigorous way, while being able to enforce my own standards of quality rather than having to dumb down and half-ass stuff because corporate said marketing said consumers said "good enough" is too complicated.

More importantly, I dabble in 14 programming languages while I hate computers, programming and high technology (aka stuff I can't repair myself with a wrench). I would still be running Windows XP if it was still supported, and I don't give a flying shit about trending techs… They are all a waste of energy and doped silicon brought to you by capitalists to keep selling useless products to people who have already all they need and more than they deserve.

I use Linux because it's the shitty OS that lets me hack it when I'm really annoyed, but I'm not an opensource evangelist and I'm firmly convinced that Linux sucks as much as Windows or Mac. It's just that Linux has a hood that can be opened with an engine that can be hacked, while the others are pretty much black boxes that work better in a large variety of cases. The fact that opensource seems to be the universal excuse for "not working" pisses me off to the maximum level and I don't believe in software developed non-professionally on saturdays.