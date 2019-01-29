https://manpages.debian.org/stretch/debhelper/debhelper.7.en.html

https://www.debian.org/doc/debian-policy/ch-opersys.html#s9.1

https://www.debian.org/doc/debian-policy/

https://www.debian.org/doc/manuals/maint-guide/dreq.en.html

https://www.debian.org/doc/manuals/maint-guide/dreq.zh-cn.html

https://www.gnu.org/software/make/manual/html_node/

https://github.com/phusion/debian-packaging-for-the-modern-developer/


Control file fields
----------------------

"Description" contains a summary on the first line, and a more verbose description on subsequent lines. The summary is what you see in apt-cache search while the more verbose description is what you see in APT GUIs such as the Ubuntu App Store or Aptitude.

The verbose description must be prefixed with a single space! And empty lines must contain a single dot character.

"Version" specifies the package version number. This consists of the application's own version number, followed by a dash, followed by a so-called Debian package revision number. The latter specifies the version number of the packaging work. Every time you modify the packaging work without modifying the application, you are supposed to bump this number but not the application's own version number. This is the first time we build a Debian package, so we specify "1".



Install deb package
---------------------

    $ sudo gdebi -n xxx.deb

    $ sudo dpkg -i xxx.deb


gdebi lets you install local deb packages resolving and installing its dependencies. apt does the same, but only for remote (http, ftp) located packages. It can also resolve build-depends of debian/control files.


See what is inside xxx.deb
--------------------------

Tool: mc

GNU Midnight Commander is a directory browser/file manager for Unix-like operating systems.

Use the arrow keys and the Enter button to browse to the .deb file in question. Then hit Enter to display the contents of the .deb file.

Can you tell me why dpkg-deb is not a proper way to make a package?

2 Building a binary package using dpkg-buildpackage
====================================================

    $ cd debian
    $ cd ..
    $ dpkg-buildpackage -b

The -b flag tells dpkg-buildpackage to build a binary package.

xxx.deb generated is on the ../ directory.

dpkg-buildpackage checks whether all Build-Dependencies (debian/control) are installed, and refuses to continue if any is missing.

There are even tools out there which will automatically install all Build-Dependencies, such as mk-build-deps. [http://manpages.ubuntu.com/manpages/xenial/man1/mk-build-deps.1.html]

Curious about what debhelper is doing? Set the environment variable DH_VERBOSE=1 and will tell you all the commands that it executes under the hood.Try:

    $ env DH_VERBOSE=1 dpkg-buildpackage -b

But it doesn't work when dh_genconcontrol substitutes these key words ${shlibs:Depends} and ${misc:Depends} with dependency like "glibc" .


3 Detailed rules to Debian package
====================================

https://www.debian.org/doc/debian-policy/

4 Filesystem Hierarchy Standard
================================

https://www.debian.org/doc/debian-policy/ch-opersys.html#s9.1


5 debian/ directory
=====================

5.1 debian/control
---------------------

* "Build-Dependency" field

Added a "Build-Dependency" field. This field specifies which packages must be installed in order to be able to build this Debian package. This is a distinct concept from package dependencies, which is what the resulting package says it will depend on. Anything you specify as a Build-Dependency is not automatically registered as a package dependency, and vice versa.

* "Architecture" field 

"all": package works on all architectures, does not require compilation

"any": package can be compiled for any Debian/Ubuntu-supported architecture

* two dependencies: ${shlibs:Depends}, ${misc:Depends}

These are magic keywords that are substituted by dh_makeshlibs and dh_gencontrol.

5.2 debian/changelog
----------------------

* time

    $ date -R

* distributions

Distributions lists one or more space-separated distributions where this version should be installed when it is uploaded; it is copied to the Distribution field in the .changes file.





6 Example
=========

* Source code

```
stephen@stephen-desktop:/data/learning_center/debian-package/debian-packaging-for-the-modern-developer/tutorial-4$ tree
.
├── debian
│   ├── changelog
│   ├── compat
│   ├── control
│   └── rules
├── hello.c
├── Makefile
└── README.md

1 directory, 7 files
```

* Build binary package

```
app@b71af44c7476:/host/tutorial-4$ env DH_VERBOSE=1 dpkg-buildpackage -b
dpkg-buildpackage: source package hello
dpkg-buildpackage: source version 3.0.0-2
dpkg-buildpackage: source distribution stretch
dpkg-buildpackage: source changed by John Doe <john@doe.com>
dpkg-buildpackage: host architecture amd64
 dpkg-source --before-build tutorial-4
 fakeroot debian/rules clean
dh clean
   dh_testdir
   dh_auto_clean
	make -j1 clean
make[1]: Entering directory '/host/tutorial-4'
rm -f hello
make[1]: Leaving directory '/host/tutorial-4'
   dh_clean
	rm -f debian/hello.substvars
	rm -f debian/hello.*.debhelper
	rm -rf debian/hello/
	rm -f debian/*.debhelper.log
	rm -f debian/files
	find .  \( \( \
		\( -path .\*/.git -o -path .\*/.svn -o -path .\*/.bzr -o -path .\*/.hg -o -path .\*/CVS \) -prune -o -type f -a \
	        \( -name '#*#' -o -name '.*~' -o -name '*~' -o -name DEADJOE \
		 -o -name '*.orig' -o -name '*.rej' -o -name '*.bak' \
		 -o -name '.*.orig' -o -name .*.rej -o -name '.SUMS' \
		 -o -name TAGS -o \( -path '*/.deps/*' -a -name '*.P' \) \
		\) -exec rm -f {} + \) -o \
		\( -type d -a -name autom4te.cache -prune -exec rm -rf {} + \) \)
	rm -f *-stamp
 debian/rules build
dh build
   dh_testdir
   dh_auto_configure
   dh_auto_build
	make -j1
make[1]: Entering directory '/host/tutorial-4'
gcc -Wall -g hello.c -o hello
make[1]: Leaving directory '/host/tutorial-4'
   dh_auto_test
 fakeroot debian/rules binary
dh binary
   dh_testroot
   dh_prep
	rm -f debian/hello.substvars
	rm -f debian/hello.*.debhelper
	rm -rf debian/hello/
   dh_auto_install
	install -d debian/hello
	make -j1 install DESTDIR=/host/tutorial-4/debian/hello AM_UPDATE_INFO_DIR=no
make[1]: Entering directory '/host/tutorial-4'
mkdir -p /host/tutorial-4/debian/hello/usr/bin
cp hello /host/tutorial-4/debian/hello/usr/bin/hello
make[1]: Leaving directory '/host/tutorial-4'
   dh_installdocs
	install -g 0 -o 0 -d debian/hello/usr/share/doc/hello
   dh_installchangelogs
	install -o 0 -g 0 -p -m644 debian/changelog debian/hello/usr/share/doc/hello/changelog.Debian
   dh_perl
   dh_link
   dh_compress
	cd debian/hello
	chmod a-x usr/share/doc/hello/changelog.Debian
	gzip -9nf usr/share/doc/hello/changelog.Debian
	cd '/host/tutorial-4'
   dh_fixperms
	find debian/hello  -print0 2>/dev/null | xargs -0r chown --no-dereference 0:0
	find debian/hello ! -type l  -print0 2>/dev/null | xargs -0r chmod go=rX,u+rw,a-s
	find debian/hello/usr/share/doc -type f  ! -regex 'debian/hello/usr/share/doc/[^/]*/examples/.*' -print0 2>/dev/null | xargs -0r chmod 644
	find debian/hello/usr/share/doc -type d  -print0 2>/dev/null | xargs -0r chmod 755
	find debian/hello/usr/share/man debian/hello/usr/man/ debian/hello/usr/X11*/man/ -type f  -print0 2>/dev/null | xargs -0r chmod 644
	find debian/hello -perm -5 -type f \( -name '*.so.*' -or -name '*.so' -or -name '*.la' -or -name '*.a' \)  -print0 2>/dev/null | xargs -0r chmod 644
	find debian/hello/usr/include -type f  -print0 2>/dev/null | xargs -0r chmod 644
	find debian/hello/usr/share/applications -type f  -print0 2>/dev/null | xargs -0r chmod 644
	find debian/hello -perm -5 -type f \( -name '*.cmxs' \)  -print0 2>/dev/null | xargs -0r chmod 644
	find debian/hello/usr/lib/x86_64-linux-gnu/perl5/5.20 debian/hello/usr/share/perl5 -type f -perm -5 -name '*.pm'  -print0 2>/dev/null | xargs -0r chmod a-X
	find debian/hello/usr/bin -type f  -print0 2>/dev/null | xargs -0r chmod a+x
	find debian/hello/usr/lib -type f -name '*.ali'  -print0 2>/dev/null | xargs -0r chmod uga-w
   dh_strip
	strip --remove-section=.comment --remove-section=.note debian/hello/usr/bin/hello
   dh_makeshlibs
	rm -f debian/hello/DEBIAN/shlibs
   dh_shlibdeps
	install -o 0 -g 0 -d debian/hello/DEBIAN
	dpkg-shlibdeps -Tdebian/hello.substvars debian/hello/usr/bin/hello
   dh_installdeb
   dh_gencontrol
	echo misc:Depends= >> debian/hello.substvars
	dpkg-gencontrol -phello -ldebian/changelog -Tdebian/hello.substvars -Pdebian/hello
	chmod 644 debian/hello/DEBIAN/control
	chown 0:0 debian/hello/DEBIAN/control
   dh_md5sums
	(cd debian/hello >/dev/null ; find . -type f  ! -regex './DEBIAN/.*' -printf '%P\0' | LC_ALL=C sort -z | xargs -r0 md5sum > DEBIAN/md5sums) >/dev/null
	chmod 644 debian/hello/DEBIAN/md5sums
	chown 0:0 debian/hello/DEBIAN/md5sums
   dh_builddeb
	dpkg-deb --build debian/hello ..
dpkg-deb: building package `hello' in `../hello_3.0.0-2_amd64.deb'.
 dpkg-genchanges -b >../hello_3.0.0-2_amd64.changes
dpkg-genchanges: binary-only upload (no source code included)
 dpkg-source --after-build tutorial-4
dpkg-buildpackage: binary-only upload (no source included)
app@b71af44c7476:/host/tutorial-4$ exit
exit
```

* See what debhelper does for us

```
stephen@stephen-desktop:/data/learning_center/debian-package/debian-packaging-for-the-modern-developer/tutorial-4$ cat debian/hello.debhelper.log 
dh_auto_configure
dh_auto_build
dh_auto_test
dh_prep
dh_auto_install
dh_installdocs
dh_installchangelogs
dh_perl
dh_link
dh_compress
dh_fixperms
dh_strip
dh_makeshlibs
dh_shlibdeps
dh_installdeb
dh_gencontrol
dh_md5sums
dh_builddeb
dh_builddeb
```

* See other generated files

```
stephen@stephen-desktop:/data/learning_center/debian-package/debian-packaging-for-the-modern-developer/tutorial-4$ cat debian/files 
hello_3.0.0-2_amd64.deb devel optional

stephen@stephen-desktop:/data/learning_center/debian-package/debian-packaging-for-the-modern-developer/tutorial-4$ cat debian/hello.substvars 
shlibs:Depends=libc6 (>= 2.2.5)
misc:Depends=

stephen@stephen-desktop:/data/learning_center/debian-package/debian-packaging-for-the-modern-developer/tutorial-4$ ls
debian  hello  hello.c  Makefile  README.md
stephen@stephen-desktop:/data/learning_center/debian-package/debian-packaging-for-the-modern-developer/tutorial-4$ tree
.
├── debian
│   ├── changelog
│   ├── compat
│   ├── control
│   ├── files
│   ├── hello
│   │   ├── DEBIAN
│   │   │   ├── control
│   │   │   └── md5sums
│   │   └── usr
│   │       ├── bin
│   │       │   └── hello
│   │       └── share
│   │           └── doc
│   │               └── hello
│   │                   └── changelog.Debian.gz
│   ├── hello.debhelper.log
│   ├── hello.substvars
│   └── rules
├── hello
├── hello.c
├── Makefile
└── README.md

8 directories, 15 files
```


7
=============

Debian packages are supposed to be built as root.
