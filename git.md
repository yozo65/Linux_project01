               
 ####  # ##### 
#    # #   #   
#      #   #   
#  ### #   #   
#    # #   #   
 ####  #   #   
               
Package: git 
Version: 1:2.25.1-1ubuntu3.2 
Priority: optional 
Section: vcs 
Origin: Ubuntu 
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com> 
Original-Maintainer: Jonathan Nieder <jrnieder@gmail.com> 
Bugs: https://bugs.launchpad.net/ubuntu/+filebug 
Installed-Size: 36,5 MB 
Provides: git-completion, git-core 
Depends: libc6 (>= 2.28), libcurl3-gnutls (>= 7.56.1), libexpat1 (>= 2.0.1), libpcre2-8-0 (>= 10.22), zlib1g (>= 1:1.2.0), perl, liberror-perl, git-man (>> 1:2.25.1), git-man (<< 1:2.25.1-.) 
Recommends: ca-certificates, patch, less, ssh-client 
Suggests: gettext-base, git-daemon-run | git-daemon-sysvinit, git-doc, git-el, git-email, git-gui, gitk, gitweb, git-cvs, git-mediawiki, git-svn 
Breaks: bash-completion (<< 1:1.90-1), cogito (<= 0.18.2+), dgit (<< 5.1~), git-buildpackage (<< 0.6.5), git-core (<< 1:1.7.0.4-1.), gitosis (<< 0.2+20090917-7), gitpkg (<< 0.15), gitweb (<< 1:1.7.4~rc1), guilt (<< 0.33), openssh-client (<< 1:6.8), stgit (<< 0.15), stgit-contrib (<< 0.15) 
Replaces: git-core (<< 1:1.7.0.4-1.), gitweb (<< 1:1.7.4~rc1) 
Homepage: https://git-scm.com/ 
Task: server, cloud-image, lubuntu-desktop 
Download-Size: 4 554 kB 
APT-Manual-Installed: yes 
APT-Sources: http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages 
Description: fast, scalable, distributed revision control system 
 Git is popular version control system designed to handle very large 
 projects with speed and efficiency; it is used for many high profile 
 open source projects, most notably the Linux kernel. 
 . 
 Git falls in the category of distributed source code management tools. 
 Every Git working directory is a full-fledged repository with full 
 revision tracking capabilities, not dependent on network access or a 
 central server. 
 . 
 This package provides the git main components with minimal dependencies. 
 Additional functionality, e.g. a graphical user interface and revision 
 tree visualizer, tools for interoperating with other VCS's, or a web 
 interface, is provided as separate git* packages. 
 
## Структура пакета
```
.
├── depacking.sh
├── git.md
└── temp1
    ├── dep_file
    │   ├── etc
    │   ├── usr
    │   └── var
    ├── dep_info
    │   ├── conffiles
    │   ├── control
    │   ├── md5sums
    │   ├── postinst
    │   ├── postrm
    │   ├── preinst
    │   └── prerm
    └── git_1%3a2.25.1-1ubuntu3.2_amd64.deb

6 directories, 10 files
## Файл preinst
```
#!/bin/sh
set -e

# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/bash_completion.d/git 1:1.8.0-1~ -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/emacs/site-start.d/50git-core.el 1:1.7.4.1-2~ -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/hooks ../../../git-core/contrib/hooks 1:1.7.7-1 -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/emacs ../../../git-core/emacs 1:1.7.4~rc1-0.1 -- "$@"
# End automatically added section


# /var/cache/git/ -> /var/lib/git/ transition
if test "$1" = upgrade &&
   dpkg --compare-versions "$2" lt-nl '1:1.8.4~rc0-1'; then
	mkdir -m 755 -p /var/lib/git
	(
		cd /var/lib/git
		for target in ../../cache/git/*; do
			if ! test -L "$target" && ! test -e "$target"; then
				continue
			fi

			link=${target#../../cache/git/}
			if ! test -L "$link" && ! test -e "$link"; then
				ln -s "$target" "$link"
			fi
		done
	)
fi

# A previous version of the /var/lib/git/ transition code
# left behind a symlink '/var/lib/git/*' -> '../../cache/git/*'.
if test "$1" = upgrade &&
   dpkg --compare-versions "$2" eq '1:1.8.4~rc0-1' &&
   test -L '/var/lib/git/*'; then
	target=$(readlink '/var/lib/git/*')
	if test "$target" = '../../cache/git/*'; then
		rm -f '/var/lib/git/*'
	fi
fi

# Git versions before 1.7.7-2 kept about 100 hard links to
# /usr/lib/git-core/git at /usr/lib/git-core/git-* to avoid
# wasting time resolving a symlink when old scripts call "git
# foo" as git-foo.  Btrfs doesn't like to have more than 130 or
# so links to a single inode in a given directory.  dpkg versions
# 1.16.1 and later temporarily double the number of hard links to
# an inode when upgrading a package.
#
# Replace the hard links with symlinks _before_ upgrading to
# avoid trouble.
#
# For added fun, coreutils mv will not replace a file by a
# symlink to the same inode (bug #654666).  We give
# /usr/lib/git-core/git its own inode to work around that.

if test "$1" = upgrade &&
   dpkg --compare-versions "$2" lt-nl '1:1.7.7-2'; then
	refinode=$(stat -c%i /usr/lib/git-core/git-add)

	rm -f /usr/lib/git-core/git.tmp
	cp -p /usr/lib/git-core/git /usr/lib/git-core/git.tmp
	mv -f /usr/lib/git-core/git.tmp /usr/lib/git-core/git
	for f in /usr/lib/git-core/*; do
		test "$f" != /usr/lib/git-core/git &&
		test "$f" != /usr/lib/git-core/git-add || continue
		rm -f "$f.tmp"
		inode=$(stat -c%i "$f")
		test "$inode" = "$refinode" || continue
		ln -s git "$f.tmp"
		mv -f "$f.tmp" "$f"
	done
fi
## Файл postinst
```
#!/bin/sh
set -e

# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/bash_completion.d/git 1:1.8.0-1~ -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/emacs/site-start.d/50git-core.el 1:1.7.4.1-2~ -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/hooks ../../../git-core/contrib/hooks 1:1.7.7-1 -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/emacs ../../../git-core/emacs 1:1.7.4~rc1-0.1 -- "$@"
# End automatically added section


test "$1" = configure || exit 0

removed_conffile=/etc/emacs/site-start.d/50git-core.el

# Carry over modifications so git-el can use them.
if dpkg --compare-versions "$2" lt '1:1.7.4.1-2~' &&
   ! test -e "$removed_conffile" &&
   test -r "$removed_conffile".dpkg-bak; then
  mv "$removed_conffile".dpkg-bak "$removed_conffile"
fi
## Файл prerm
```
#!/bin/sh
set -e
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/emacs ../../../git-core/emacs 1:1.7.4~rc1-0.1 -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/hooks ../../../git-core/contrib/hooks 1:1.7.7-1 -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/emacs/site-start.d/50git-core.el 1:1.7.4.1-2~ -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/bash_completion.d/git 1:1.8.0-1~ -- "$@"
# End automatically added section
## Файл postrm
```
#!/bin/sh
set -e
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/emacs ../../../git-core/emacs 1:1.7.4~rc1-0.1 -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper dir_to_symlink /usr/share/doc/git/contrib/hooks ../../../git-core/contrib/hooks 1:1.7.7-1 -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/emacs/site-start.d/50git-core.el 1:1.7.4.1-2~ -- "$@"
# End automatically added section
# Automatically added by dh_installdeb/12.10ubuntu1
dpkg-maintscript-helper rm_conffile /etc/bash_completion.d/git 1:1.8.0-1~ -- "$@"
# End automatically added section
