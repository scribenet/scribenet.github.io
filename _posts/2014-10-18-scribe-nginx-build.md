---
layout: post
title: Scribe Nginx Build
---

Over the past few months, we'd found ourselves building custom Nginx packages by hand regularly with
each new [Ubuntu Nginx Mainline](http://ppa.launchpad.net/nginx/development/ubuntu) release, and it
was becoming quite tedious. Having never worked with [Launchpad](http://launchpad.net) prior, we began
our journey by simply writing a script that would handle building a reliable Debian package for our own 
internal use.

After additional time working in this manner, it became obvious that for us to maintain a consistent 
environment across our development workstations and our production servers, we would need to investigate
the basics of Launchpad. Soon afterward, the [scribeinc](launchpad.net/~scribeinc) Launchpad was created,
including our first PPA for [nginx](https://launchpad.net/~scribeinc/+archive/ubuntu/nginx).

### Launchpad

So as not to duplicate the content already well-written on the by the Laucnhpad volunteers, we aren't 
going to go into the specifics of setting up Launchpad, adding trusted keys, or any of the basic procedures
involved in getting your initial account setup. For that, the folks maintaining the [Getting Started Documentation](https://help.launchpad.net/Packaging/SourceBuilds/GettingStarted)
have done a wonderful job.

Instead, we're going to take a moment to describe our initial missteps, and how our build script evolved 
to support automatic local sandbox building/testing, followed by the upload of the relevant source and 
diff files to Launchpad that ultimately cause their builders to create our final Debian packages.

### SimpleSBuild

Our initial attempts to get our uploaded files to build properly on Launchpad failed miserably. At this 
point we were simply testing the build using the `dpkg-buildpackage` command we had used in the past to 
create the Debian package distributed to our developers and later pushed to our production environment.
Getting assistance on what we were doing wrong turned out to be exceptionally easy: joining the `#launchpad`
channel on IRC (Freenode Server).

Once we started asking some basic questions, our issue became clear. Using `dpkg-buildpackage` didn't 
provide a "clean" environment for our build to run; it allowed our build to reply on the underlying 
packages and setup of our specific Ubuntu installation to compile successfully. This enviornment wasn't 
anywhere near the same as that of the automated package builders dispatched by Launchpad when source files
were uploaded.

We were directed to [SimpleSbuild](https://wiki.ubuntu.com/SimpleSbuild). Following the directions outlined
in that help document allowed us to setup a function build environment that that we could test our packages
against prior to uploading to Launchpad. While not an exact replica of the automated build agents on Launchpad,
such an environment was extremely close.

After configuring SimpleSbuild and attempting to compile our package within that environment locally, 
the issues with out build process quickly became apparent, allowing us to amend our procedure to suite
the enviornment it would be build against to create our final PPA Debian packages.

### Newest Build Script

Our latest build script allows for compiling in additional modules with ease. At this time, our build of
Nginx is statically compiled against the latest (at this time v8.36) release of PCRE to support JIT regex
parsing within our configuration files.

At this time, we build Nginx with the following flags

`--with-debug`, `--with-pcre-jit`, `--with-ipv6`, `--with-file-aio`, `--without-poll_module`, `--without-select_module`, `--without-http_uwsgi_module`. 

and the following modules

`http_ssl_module`, `http_stub_status_module`, `http_realip_module`, `http_auth_request_module`, `http_geoip_module`, `http_spdy_module`, `ngx_pagespeed-release-beta`, `alphashack_nginx_graphdat`, `nginx-auth-pam`, `nginx-development-kit`, `ngx-fancyindex`, `nginx-upload-progress`.

### Using our PPA

Using our PPA is as simple as running the following on any supported Ubuntu release (currently Trusty and Utopic):

```bash
sudo apt-add-repository ppa:scribeinc/nginx
```

### Customizing

If you'd like to build your own packages, feel free to clone our build repository:

```bash
git clone https://github.com/scribenet/nginx-scribe.git
```

Be sure to open up the `bootstrap.sh` file and edit the beginning configuration variables to suite your needs. 

Once you're ready to give it a try, assuming you have SimpleSbuild configured as described in the aforementioned documentation, you
can try building your package using `BUILD_MODE=4` to attempt a build locally within a sanitized sbuild environment.

Have question or comments? Please let us know [on GitHub](https://github.com/scribenet/nginx-scribe/issues)!

--- 
Rob M Frawley 2nd, *Systems Architect of Scribe Inc.*