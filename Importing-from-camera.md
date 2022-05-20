Upstream darktable has a tethering view and a way to mount camera file system (SD or CF memory cards) through USB, using the library GPhoto2.

But the desktop environment is also be able to mount the camera as a local hard drive using PTP and/or MTP protocol, possibly also through Gphoto2.

The problem in upstream darktable is then that the desktop environment and darktable may compete to access the camera file system through GPhoto2, leading to inconsistencies and weird behaviours. The simplest way (that also works on Windows) is to let the operating system or desktop environment mount the camera file system as a storage media, and then access it in darktable as just another directory. 

For this reason, the support of GPhoto2 has been entirely removed from R&Darktable. 

## How to import photos from a camera ?

### Easy and fast

The most straightforward way is simply to plug your SD card into the SD reader of your computer, if any. This removes the need to use any intermediate driver and software stack that has many opportunities to fail, but also provides typically the best transfer rates.

### Possibly difficult

If you don't have an SD card reader, or if your camera is using other types of memory cards (CF, XQD), you need to plug in your camera to your computer using an USB cable. To read the content of the memory card, your computer will need to have the proper drivers installed. On Linux, you may need to install one or more of the following packages :

* libmtp,
* linuxptp or libptp,
* libgphoto2 and possibly gphoto2 (which includes libptp),
* kio-mtp + kio-extras on KDE environment, or gvfs-mtp + gvfs-gphoto2 on Gnome environment.

In any case, those methods can still fail because of udev rules and so on. These methods will always be at best brittle. Buying a card reader will spare you a considerable amount of time and problems by removing the need to connect to disputable drivers.


## The state of tethering on Linux

Tethering looks pro and sounds cool. When it works.

Tethering on Linux happens through a single library : GPhoto2. After having tested it several times since 2013, with 3 different cameras, I came to the conclusion that it is nowhere near usable in a production environment. Sure, you will always find that geek that spent 3 days debugging all the stack to be able to say "I did it", or that guy who is lucky to have the right camera and the right version of the software. But the truth is it's totally random, even on cameras supposed to be fully supported by GPhoto2 and I have fully-supported cameras that worked in the past but don't on more recent versions.

The rule of thumb is you will spend as much time trying to make GPhoto2 work than doing actual photography, which is not good for your credibility if you work for, with and in front of clients. The worst is you might have to troubleshoot again when updates come. Simply get a Windows box and buy a decent tethering soft with user support.

Not to mention, the main reason for doing tethering on the desktop is to better see the pictures on a larger screen. But the live view that GPhoto2 provides has the same resolution as the back screen of the camera anyway, so it's just bigger pixels.