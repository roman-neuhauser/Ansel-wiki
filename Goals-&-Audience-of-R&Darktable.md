# Goals

## Introduction, Context

Digital photography has widely spread in the 2000's as it allowed faster workflows and made possible to get immediate results, compared to the traditional film & darkroom workflow. But this introduced many new problems.

Firstly, analogue photographers were not necessarily accomplished lab and printing technicians, but could rely on local photo labs to get their developments and prints done. Digital photography added the burden of processing the "digital negatives" (raw files) on the shoulders of the photographers, by means of software. But those photographers didn't usually got the appropriate training, both in digital colorr management and in general computer use. That pushed many of them in the hands of over-simplified software, verging on the toy side, which defined the mainstream expectation of what a digital photo editing software should be.

Secondly, the analogue printing is a (mostly) What You See Is What You Get (WYSIWYG) process, despite its complexity and need for real skills, where the lab tech interacted directly with the finished product : the print (granted, the print did not show an image before development). Digital processing is not WYSIWYG because the monitor used to preview the result will generally not have the same visual properties as the printing medium or as the consumer display. This introduces complexity in the imaging pipeline that needs to be handled through the [model-view-controller paradigm](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), something that many software did not get right. Display profiling workflows, like the ICC framework, have tried to automatize and hide most of the complexity of color management, but they mostly managed to confuse users even more by abstracting too much things that can actually be written as a couple of equations using only domestic-accounting-level maths (sums and products).

Thirdly, the analogue lab tech could directly (physically) access the paper and the negative to perform all kinds of adjustments and even distortions (dodging & burning, masking, split-toning, cross-processing, bleach bypass, solarization, etc.). This direct physical access enabled artists to hack the printing process in ways sometimes unplanned by vendors, to match their targeted visual outcome. Digital imaging has cast away the techs and artists alike from the medium, and they are confined to the functionality provided in the GUI of their software. The actual image lives in the computer as pure data, and only engineers know how to access it and what happens to it.

Finally, digital imaging has been driven by computer scientists rather than by color scientists, let alone by actual photographers or photo lab technicians. They developed a digital language incompatible with the legacy of analogue photography and collections of editing tools making use of anonymous unit-less parameters that didn't care for optics and exposure values, thus confusing the analogue-trained photographers (used to drive brightness through light exposure or chemical developing times…). Their constant reliance on "barely working" color spaces (HSL, non-linearly encoded RGB) to handle color changes impregnated many an user with bad habits which soon became expectations, then requirements. The color pipelines developed by the coding monkeys later needed a completely different workflow to handle HDR images, because their hacky logic didn't scale with dynamic range and "worked" only as long as camera sensors and computer displays had mostly the same dynamic range as paper prints, with no future-proofing intended.

## High-level goal of R&Darktable

R&Darktable aims at being an instrument of visual expression letting artists develop their own interpretation of the raw material by allowing a large expressive range of color effects, much like a musical instrument lets musicians interpret the score by allowing a large expressive range of sound effects. This is achieved by putting back the retoucher in the center of the image processing, by allowing a transparent and as-direct-as-possible access to the image data, and by providing color manipulations making sense either on an optical level or on a psycho-perceptual level. 

## User-level goal of R&Darktable

1. Allow to efficiently cull the photographs coming out of the camera/memory card, in order to pick only the ones worthy of being fully post-processed,
2. Allow to edit/retouch the culled photographs in the most direct way, with a minimal number of steps, using unit image controls that affect only one perceptual or optical property at a time,
3. Allow indexing and later retrieval of the processed photographs for archival purposes.


# Supported workflows
 