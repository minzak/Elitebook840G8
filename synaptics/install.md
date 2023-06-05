# Final Guide for v1.94.1 on Ubuntu/Pop!_OS

All credit to @Brett_Kosinski's [original guide](https://blog.b-ark.ca/2021/09/06/debian-on-framework.html).

### libfprint 1.94.1

1. To begin, create a working directory and download the Debian package source, the libfprint source, and associated build dependencies:

```
mkdir build_dir
cd build_dir
curl -O https://deb.debian.org/debian/pool/main/libf/libfprint/libfprint_1.90.7-2.debian.tar.xz
sudo apt build-dep libfprint
sudo apt-get install libgudev-1.0-dev
git clone https://github.com/freedesktop/libfprint.git 
```
2. Now, we’ll jump into the libfprint repository, check out the 1.94.1 tag, and unpack the debian control files:
```
cd libfprint
git checkout v1.94.1
tar -xJvf ../libfprint_1.90.7-2.debian.tar.xz
```
3. To begin, we need to make some changes to the `./debian/rules` file. First, we’ll update the rules to only build the driver we need:
```
CONFIG_ARGS = \
    -Dudev_rules_dir=/lib/udev/rules.d \
    -Dgtk-examples=false \
    -Ddrivers=synaptics
```
4. We’ll also need to comment out the lines for moving around the autosuspend files (as far as I can tell they’re only needed for the ELAN driver):
```
override_dh_install:
#	mv debian/tmp/lib/udev/rules.d/60-libfprint-2-autosuspend.rules \
#		debian/tmp/lib/udev/rules.d/60-libfprint-2.rules
```
5. Next up, we need to modify `./debian/libfprint-2-2.install` to install `hwdb.d` and not `rules.d`:
```
lib/udev/hwdb.d/
usr/lib/${DEB_HOST_MULTIARCH}/libfprint-*.so.*
```
6. Next, we’ll update `./debian/changelog` to add an entry for our version at the top of the file, you can add your own info:
```
libfprint (1:1.94.1-1) unstable; urgency=medium

  * custom build for use on my Framework laptop

 -- Brett Kosinski (brettk) <email>  Fri, 6 Sep 2021 20:05:00 -0700
```
7. Build the package, it will fail:
```
sudo dpkg-buildpackage -b -uc -us
```
```
dpkg-gensymbols: error: some new symbols appeared in the symbols file: see diff output below
dpkg-gensymbols: warning: debian/libfprint-2-2/DEBIAN/symbols doesn't match completely debian/libfprint-2-2.symbols
```
8. Fix this error by copying the symbols file it is comparing against:
```
cp ./debian/libfprint-2-2/DEBIAN/symbols ./debian/libfprint-2-2.symbols
```
9. Build again and it should work this time, producing 4 .deb files in the parent directory:
```
sudo dpkg-buildpackage -b -uc -us
```
10. Install these with dpkg:
```
sudo dpkg -i ../*.deb
```
11. You can remove all the files now and uninstall the build dependencies if desired.
### frpintd 1.94.0
1. While getting libfprint built was a bit of a pain, fprintd is much simpler. The first essential steps are the same (starting in some build directory):

```
mkdir build_dir
cd build_dir
apt source --download-only fprintd
sudo apt build-dep fprintd
git clone https://github.com/freedesktop/libfprint-fprintd.git 
cd libfprint-fprintd
git checkout v1.94.0
tar -xJvf ../fprintd_1.90.9-1.debian.tar.xz
```
2. Edit `./debian/rules` to overide a check for dependency information. Add these lines to the end and make sure a tab is used rather than spaces for the spacing on the last line:
```
    override_dh_shlibdeps:
	dh_shlibdeps --dpkg-shlibdeps-params=--ignore-missing-info
```
3. Edit `./debian/changelog` to create a new entry for our version, you can add your own info:
```
fprintd (1:1.94.0-1) unstable; urgency=medium

  * custom build for use on my Framework laptop

 -- Brett Kosinski (brettk) <email>  Fri, 6 Sep 2021 20:05:00 -0700
```
4. Then build the package!
```
sudo dpkg-buildpackage -b -uc -us
```
5. Install the created .deb files:
```
sudo dpkg -i ../*.deb
```
6. Enable and start the service:
```
sudo systemctl enable fprintd.service
sudo systemctl start fprintd.service
```
7. Activate the PAM module.
```
sudo pam-auth-update --enable fprintd
```
8. You can remove all the files now and uninstall the build dependencies if desired.
### Enrolling Fingerprints
Setting up fingerprints

At this point, if all went well, assuming you’re using Gnome you can enroll a new set of fingerprints:

1. Hit the Windows key, type “Users”, and open the settings panel
2. Select “Fingerprint login”
3. Enroll fingerprints

Voila! At this point all of the hardware on your laptop should be functioning.
    Voila! At this point all of the hardware on your laptop should be functioning.

[quote="RandomUser, post:132, topic:1501"]
…now if someone would be kind enough to collect all that into a single script (e.g. If distro = “Ubuntu” AND motherboard = “FRAMEWORK” AND current libfprint version < 1.94 THEN…)
[/quote]

It's really not a hard process, but here's a dumb script that should get the job done. It's not smart enough to edit the files, so it just deletes them and pulls edited copies from my webserver. This is very unsafe so if you can't read and understand this script (also verify any files it downloads) you should NOT be running it!

[ubuntu_libfprint_v1.94.1_install.sh](https://drop.azokai.com/TFGGQoLL/ubuntu_libfprint_v1.94.1_install.sh)

`bash ubuntu_libfprint_v1.94.1_install.sh`

(Tested on Pop!_OS, but i think it should work fine on Ubuntu)
