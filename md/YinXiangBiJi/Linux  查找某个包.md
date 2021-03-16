Linux  查找某个包

Linux  查找某个包

Looks like libnspr4-dev is the package you want.
I found it using apt-cache search:

	$ **apt-cache search nspr**
	     dnsproxy - proxy for DNS queries
	     libnspr4 - NetScape Portable Runtime Library
	     libnspr4-0d - NetScape Portable Runtime Library - transitional package
	     libnspr4-dbg - Debugging symbols for the NetScape Portable Runtime library
	     libnspr4-dev - Development files for the NetScape Portable Runtime library
	     python-nss - Python bindings for Network Security Services (NSS)

 Looks like it's already in the Debian repositories, which means you don't need to get it from elsewhere.

You can see which version would be installed with apt-cache policy:

	$ **apt-cache policy libnspr4-dev**
	libnspr4-dev:
	  Installed: (none)
	  Candidate: 2:4.9.2-1
	  Version table:
	     2:4.9.2-1 0
	        500 http://ftp.uk.debian.org/debian/ testing/main amd64 Packages

If that's OK, you can install it with `apt-get install libnspr4-dev`.

If not, then you should check `/etc/apt/sources.list` to ensure that it lists a repository with the version you need.