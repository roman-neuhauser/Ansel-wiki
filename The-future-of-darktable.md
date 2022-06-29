It's funny to see how people reacted to my fork and then to vkdt by asking questions about the future of darktable.

# Backstory

I started using darktable at version 0.9 circa 2010-11. I found it featured on Playdeb.net, a website specialized in reviewing (the very few) video games (available) for Linux (at that time) and providing Debian repositories with fresh packages, and also had a Getdeb counterpart for desktop apps. Completely random. Until then, I had struggled with UFRaw, Digikam and Rawtherapee with little success.

In the early 2010's, raw photographs had 12-16 Mpx and a crappy dynamic range that happened to be on-par with the one of the typical desktop display. darktable was designed for this : to work on CPU, with Gtk, on 10-12 MB files having a reduced dynamic range.

As of 2022, it is still the only Linux photo editor that supports natively GPU hardware acceleration through OpenCL. But what was a luxury 10 years ago is now a requisite with 45-54 Mpx raw photographs, especially since usage got more demanding with heavier denoising, deblurring, missing parts reconstruction, masking and mask feathering.

Not to mention, the raw files now have a dynamic range almost twice as large as the typical desktop monitor, which puts more pressure on the internal color science regarding color fidelity and predictability through intensive tone mapping. 

Digikam, Rawtherapee, Lightzone and the others still haven't upgraded neither for GPU nor HDR processing abilities.

# Early design choices: 2009-2013

Without going in all the tedious details, darktable image processing features are organized around a modular pipeline where pixels are filtered sequentially through "modules" or "image operations" (IOP). Those can be seen as adjustment layers in raster editors like Photoshop, especially since they get an alpha (occlusion) channel and blending modes to perform basic compositing within the same image. Modules are self-enclosed, don't really see each other, and are driven from the pipeline. They also contain both the pixel code and the GUI code of the widget in which they appear in the UI.

All IOP's pixel code is at least written in plain C99. Those make use of the OpenMP library for parallelism and multi-threading. Many IOP have also a redundant code path written with Intel SSE2 intrinsics, to take advantage of the SIMD instructions (vectorized) introduced with Intel Pentium 4. The SSE2 code is hard to read for non-specialists, even though it's still better than pure assembly, and you need the vanilla C99 to be able to debug and understand what it is doing. Then, some IOP have an OpenCL kernel, doing exactly the same thing as the C99 and SSE2 pathes but in another language. 

Since all 3 pathes have different memory patterns and strategies, the code is not always translatable directly, sometimes the logic needs to be adapted to the memory layout for better performance. Also, GPU and CPU arithmetic are slightly different, especially for hardware-accelerated (and vendor-dependent) operations. Asserting the consistency of the image filters outputs, through all 3 code pathes, is a nightmare in itself. Integration tests, checking the output of OpenCL and C on test images, were introduced only circa 2019.

The choice of OpenCL is not great, but there was not much else available at the time. Indeed, it is intended for numerical computation (like data crunching), and not for user interaction (that would be OpenGL). The logic of OpenCL is to "offload" a computational plan from the CPU to the GPU, meaning everything is controlled from and by the CPU, including buffer copies between system RAM and video RAM — big performance bottleneck.

Since your HDMI socket is soldered on the GPU chip, it might make sense to fetch the picture straight on GPU and to send it directly to display, but OpenCL doesn't allow it. Anyway, the GUI is painted through Gtk, which works on CPU only, so the rendered image has to come back from GPU/OpenCL to the system RAM to be painted in the window by Gtk/CPU, only to be sent again to the GPU because… your HDMI socket is soldered on it anyway. Bottlenecks everywhere, but how to do this differently at the time ?

# From usable to complete: 2013-2018

darktable had a parametric masking system already, where each module behaved like a layer and where the mask would be applied as an alpha channel to blend the output of the module over its input. This was extended circa 2013 to drawn masks, a much praised feature at the time, but that came at the price of bending the pipeline in a way it hadn't been planned for. 

Drawn masks using vector drawings defined in the system of coordinates of the raw file, to this day, you still can't draw a straight gradient mask if there is a lens correction for the cushion effect. That is, when the user defines a control point for a drawn shape, the points coordinates in user space (relative to the screen) are translated into raw space (going through the reverse perspective and lens correction transforms), and the mask is then rendered through the direct transforms. The problem of this approach is geometric shapes like circles (created from a center node and a radius) can't go through a geometric transform, only their nodes can, but then when the user draws a circle, the result he gets is an ellipse. The hack that was found in 2021 is to partially rasterize them and then transform them, meaning every brush stroke has to go through several useless computations and the final mask position will anyway not be pixel-accurate for all zoom levels, depending on the distortions (mostly perspective and lens correction).

The drawn mask extension was controversial at the time, among developers, specifically because the code was brittle and hacked in-place, understandably to avoid massive architecture changes, but with slow inefficient code, and under disputable made-up constraints (favouring mask re-editability in a non-destructive fashion over pixel-correctness and consistency with user input — *drawing a circle should not produce an ellipse, linear gradients should not be bent*). This also raises the question of how/by whom the constraint of having vector masks designed for maximum re-editability was enforced, where raster masks à la Lightroom/Capture One are much faster to set-up and seem to require very little later adjustments, while removing on intermediate step in the pipeline (because vector masks will need to be rasterized anyway before application).

This was one of the first steps of a twister game in darktable's development, where new features would need increasing care to avoid interference with others through logics less and less uniform and systematic. The next was multi-instantiation of modules, which triggered some undefined behaviours : the darkroom histogram being wired to exposure/black controls, to which exposure module instance should it be wired ? And what about keyboard shortcuts ? Which instance should capture them ? This was only "solved" circa 2020-21 through an user preference.

# From display-referred to scene-referred : 2018-2022

The darktable's pipeline had initially been thought to be modular and re-orderable. On second-thought, the re-orderable part was abandonned and modules where placed in a static order, decided at compilation time. At the same time, to enforce the compatibility of older edits, the pre-existing modules needed to be forced in the same order as before when new modules were added. So, the graph solver supposed to flatten all constraints became a liability more than an helper, and modules had to be inserted in pipeline manually.

This fixed pipeline enabled some fixed assumptions inside modules and in the whole pipeline regarding what modules came before and after, namely regarding the distorting modules (crop, rotate, perspective, lens correction).

The scene-referred scheme imposed legacy, display-referred, modules to be sent at the end of the pipeline where they belonged. In a fixed pipeline, that would have requested to create duplicate modules, wire them where they belonged, and deprecate the previous. But then, some modules could have different and equally valid positions (LUTs). So the pipeline was made flexible, and modules were allowed to be moved, therefore associated with a priority. But then, the fixed assumptions needed to be made explicit, which uncovered a lot of silent bugs in the process.

To this day, copy-pasting history stacks or applying styles, especially when multiple instances are used, may end in garbled pipeline order, but it's totally random and hardly reproduceable. 