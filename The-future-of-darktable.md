It's funny to see how people reacted to my fork and then to vkdt by asking questions about the future of darktable, while frankly, nothing has changed and darktable's future has anyway never been guaranteed.

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

The darktable's pipeline had initially been thought to be modular and re-orderable. On second-thought, the re-orderable part was abandonned and modules where placed in a static order, decided at compilation time. At the same time, to enforce the compatibility of older edits, the pre-existing modules needed to be forced in the same order as before when new modules were added.

This fixed pipeline enabled some fixed assumptions inside modules and in the whole pipeline regarding what modules came before and after, namely regarding the distorting modules (crop, rotate, perspective, lens correction).

The scene-referred scheme imposed legacy, display-referred, modules to be sent at the end of the pipeline where they belonged. In a fixed pipeline, that would have requested to create duplicate modules, wire them where they belonged, and deprecate the previous. But then, some modules could have different and equally valid positions (LUTs). So the pipeline was made flexible, and modules were allowed to be moved, therefore associated with a priority. But then, the fixed assumptions needed to be made explicit, which uncovered a lot of silent bugs in the process.

To this day, copy-pasting history stacks or applying styles, especially when multiple instances are used, may end in garbled pipeline order, but it's totally random and hardly reproduceable. These problems stem in the loss of generality (and a bit of lazyness) allowed when the early project of re-orderable pipeline was abandoned. 

# Looking at the future through the past

If we summarize, darktable has a major performance bottleneck through OpenCL and Gtk. Yes, OpenCL allows to unleash the power of the GPU, but it has an incompressible overhead of memory buffers copies. This is grounded in its architecture and will not be solved by another hack-in-place. The magnitude of the requested rewrite overcomes starting again fresh, not to mention the phase of instability it would init (we still haven't fully recovered from the pipe reordering yet…).

A software is a tool. A tool is more than the sum of its functionalities. A Leatherman multi-tool or a Swiss Army knife are packed with functionalities, so why hasn't every handyman replaced his whole toolkit with a simple multi-tool ? Well, they are bulky, their ergonomics are not great, they are fragile and can't handle high torque on the screwdrivers… all in all, they aren't really practical for accurate work or regular extended use. Helpful in camping, but they won't replace full-handle tools.

Usability is an overlooked concern, and runtimes contribute directly to it. Barely working is not enough. Usability answers a simple question: can you use this tool comfortably for a long amount of time and meet your productivity requirements ?

Modules like diffuse or sharpen have pushed the darktable's pipeline to its limits. And this is only mild supervised machine learning. What good would it do to keep loading this old beast ?

But the most blocking thing is the non-destructive nature of darktable's work, which means "edits" are nothing but a cooking recipe until a program renders a base material applying this. Which means old recipes need to be rendered today the same as 10 years ago. This legacy to carry gives little room to cleanly fix core design mistakes, at most it ends into more heuristics trying to handle the old case and the new one, so more code, more burden, more bugs, less stability.

Something has to give, all these constraints together are simply too much. darktable has been pushed already farther than what its initial design allowed, in ways that have introduced architectural brittleness and a patchwork of ugly fixes. Moving from OpenCL to Vulkan, and replacing Gtk, and keeping compatibility with the youth mistakes of darktable from the 2010's is too much to ask from anyone. 

# What is development ?

Behind the question "will darktable continue to be developed ?", I feel there is something misunderstood. A tool is developed until it fully fills its purpose. What is missing today in darktable ? 

See, adding ever more features, in a true capitalistic powertrip, is not improving. At some point, when your trays, shelves, tables and stands are covered with the most accurate and specialized tools, any additional productivity brought individually by each of these tools gets lost in the time you waste chasing them through your workshop every time you need one. Rather keep their number under control, stick to general and all-purpose tools that complement each-other, and keep your tables for working instead of storing garbage.

If you keep on forced march, past feature-completeness, you end up making a toy, packed with exciting new features, that will not make you more productive but may well waste your time chasing features you need through the lot of useless ones swamping the GUI.

Development is actively damaging stability and induces more maintenance work. It's literally work that will give more work. So development for the sake of development, or engineering for the sake of engineering, is madness and does users a disservice too.

So, let's not mistake development with maintenance.

# The future of photography

It's funny to ask questions on the future of a tool without having first a good global look at photography as a discipline.

In 2010, raw photographs had 12-16 Mpx and dynamic ranges around 8 EV. In 2022, they casually blast 45-54 Mpx and 14 EV at base ISO.

Photo editing software have been split into the "virtual darkrooms" (Lightroom, Capture One, darktable) and the "retouching soft" (Photoshop, Gimp, Krita). Those lines have been blurred over the years. Actually, the boundary between developing and retouching is now entirely arbitrary, the tools are pretty much the same. Although the GUIs are still firmly different, and lots of photographers make it a principle to not "photoshop" their pictures (yes, it's a verb). Lately, the "AI"-driven toy software have also appeared, allowing quick and mainstream manipulation of content up to creating deep-fakes. 

The film stocks looks have been replaced in digital by presets, color profiles (more or less opinionated). Only at the end of the 2020's, photographers started to get inspired by movies and work their color palette as a visual language. It took until 2020 or 2021 to Lightroom to introduce the color balance, similar to the star feature of Capture One, for color grading (don't tell them, but darktable had it since at least 2016). 

On the other end, I think it's fair to say that smartphones have won the battle of the consumer/entry-level photography. We don't need to care about those anymore, they will never shoot raw. But among the raw shooters, there are still a bunch who make it a priority to open a complicated software only to get the exact same look as the out-of-camera JPEG, but photography being a sport of equipment, there is a status component and you need to be a raw shooter in some circles.

I wish to digital photography to become more than just a metaphor of analog, to embrace the digital medium to blur even more the boundaries between painting, drawing, still images, movies, to include sound, 3D renders, other images. All these technologies are workable within a computer, so they are all unified in the silicon box. 

Being able to compose raw pictures directly in scene-referred is a must. As is painting over photographs in a scene-referred workflow, because it's the only simple and clean way to handle HDR without having to treat it like a disabled citizen with particular needs.

I also wish it was easier for artists to create procedural images through scripting, like Brownian motion backgrounds or fractals, without being all the time dependent on what developers are willing to code them. Regular people are now playing with Arduino and Raspberry Pi for wearables and artistic projects, scripting procedural visual is very much in line with this.

# About digital assets management (DAM)

Digital assets management is the process of maintaining an archive of pictures in a way that allows efficient access and use. Unfortunately, use cases and needs vary greatly, and having discussed these matters extensively for 5 years with various crowds, everybody wants efficient DAM but nobody is able to say what it should do and how it should look.

DAM also overlaps with the culling process, which consists in selecting the body of pictures that will get post-processing attention. 

It's only lately with a couple of converging opinions, from old-time Adobe users, that I came to the conclusion that DAM doesn't need to be anything more than file management extended with XMP and EXIF support. The least it does, the better. It's just about being able to show a grid of thumbnails and run queries through files to search for tags or metadata. Then, let it be scriptable for niche uses and plugins.

But it seems that hardcore professionals have actually simple DAM needs, in fact much simpler than most amateurs, who have too much time on their hands to Hermione Granger the book-keeping with never-ending tagging.

Abstracting file management through database allows users to shoot themselves by dumping all pictures in the same folder and index their pictures in virtual collections stored only in the database. That traps the user in the software and induces a high risk of data loss if any DB corruption happens. On the other hand, a sane (sub)-directory structure is application-agnostic and uses a very basic block of any computer.

Workflow-wise, the post-processing app only needs to have a sort of bookmarking to keep track of which pictures are pending retouch, which are done, which have been exported, etc.

To create "collections" (as Lightroom calls them), the minimalistic way is to assign a certain tag to pictures, and then query that tag when needed. 