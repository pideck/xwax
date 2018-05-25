# Building xwax for the PiDeck

This branch contains GUI modifications for running XWax on
a small touchscreen (i.e. a 7" Tablet)

## How to compile and test xwax GUI in a test system

./configure --prefix /usr --enable-alsa --enable-jack
make clean; make EXECDIR=.
./xwax -i scan -s bpm-scan --dummy -g 630x480 --no-decor -l ~/Music

## How to build the Debian package

### Build the source package

In the xwax directory build a signed source package ignoring the .git directory,
then go up to the parent directory and find the dsc file:

```
dpkg-buildpackage -i.git -S
cd ..
ls xwax*dsc
```

You will be prompted for your passphrase.

### Set up or update pbuilder

First, set up your pbuilder environment, which is a one-time operation per build machine...

1. Install the essential packages:

`sudo apt install pbuilder ubuntu-dev-tools`

2. Create chroot environments for the first target distro on all supported architectures. For example: 

`pbuilder-dist buster armhf create`

The base.tgz packages will be created under your ~/pbuilder directory.

If instead you see errors about the distro name not being found or 'no such script', it's because the target distro is newer than the build distro. In this case, you need to update your distro-info-data and debootstrap packages. Debian sid has the latest versions in .deb format; you can download them manually and install these packages with the dpkg -i command.

3. Repeat step 2 for each distro and architecture you want to build for. In all they will use about 1GB to 2GB disk space if you build packages for several distros or architectures.

4. If you use the machine for building packages some time later, update each chroot environment before building with (for example):

`pbuilder-dist buster armhf update`

Repeat for the other chroot environments.

### Build the binary package

Set pbuilder working using the dsc file that you found earlier:

`pbuilder-dist buster armhf build xwax_1.7-2~pideck.dsc`

If all goes well, you will find your new package in `~/pbuilder/buster-armhf_result/`

If you get a segfault from pbuilder-satisfydepends, append this line to your ~/.pbuilderrc file:

`PBUILDERSATISFYDEPENDSCMD=/usr/lib/pbuilder/pbuilder-satisfydepends-apt`

You may see a lot of errors like this:

`qemu: Unsupported syscall: 383`

These are due to qemu being stricter about what is happening than in the past. If you ignore them, the build will continue eventually.
