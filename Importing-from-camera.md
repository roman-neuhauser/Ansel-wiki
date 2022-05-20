Upstream darktable has a tethering view and a way to mount camera file system (SD or CF memory cards) through USB, using the library GPhoto2.

But the desktop environment is also be able to mount the camera as a local hard drive using PTP and/or MTP protocol, possibly also through Gphoto2.

The problem in upstream darktable is then that the desktop environment and darktable may compete to access the camera file system through GPhoto2, leading to inconsistencies and weird behaviours. The simplest way (that also works on Windows) is to let the operating system or desktop environment mount the camera file system as a storage media, and then access it in darktable as just another directory. 

For this reason, the support of GPhoto2 has been entirely removed from R&Darktable. 