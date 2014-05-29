gcloud-compute-ssh
==================

Patches to [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) for [`gcloud compute ssh`](https://developers.google.com/cloud/sdk/gcloud/reference/compute/ssh) on Windows.

overview
========

Patches PuTTY for Windows [source code](http://tartarus.org/~simon/putty-snapshots/putty-src.zip) (snapshot taken on 5/28/2014) so that after renaming

  - `copy pscp.exe scp.exe`
  - `copy plink.exe ssh.exe`
  - `copy pkeygen.exe ssh-keygen.exe`

`scp`, `ssh` and `ssh-keygen`, from the command line, behave more like the
OpenSSH counterparts. Some behaviors are not covered -- mainly just enough to
support internal [`gcloud`](https://developers.google.com/cloud/sdk) usage patterns.

details
=======

Changes fall into a few categories:
* Change `cmdgen.c` to generate pkeygen.exe which acts like `ssh-keygen` from the
  command line. Most of the changes are in this file. Major changes:
    - use Windows libraries to generate cryptographic random data
    - output refactored to a loop that can generate more than one file
    - `id`, `id.pub` and `id.ppk` all generated by default
* Change all identity file read acceses to first check for a .ppk variant, e.g.,
  `-i id' will first check for `id.ppk`. This preserves the `ssh` and `scp -i` 
  usage pattern.
* Change the default `TERM` environment variable value passed to the server by
  `plink` to check the local `TERM` setting (instead of "xterm"). If `TERM` is not 
  set then "dumb" is used. This gives the proper hint to remote `.profiles` to
  refrain from colorizing prompts and `ls(1)` output.
* Eliminate the low hanging fruit of the MSVC 32 and 64 bit warnings. Most of
  these involve mixed combinations of `[unsigned] int`, `[unsigned] long`, `size_t`,
  and `ssize_t`. Some only show up in 64 bit compilers. Other culprits are
  `strcpy()`, `strcat()`, and `sprintf()` into fixed size buffers - they are handled
  by a homebrew `szprintf()` that supports a clean sized buffer paradigm. *Much
  more work* is needed in this area. A lot of the `(int)` and `(size_t)` casts
  should be replaced by proper variable and function typing, but that would
  require a meticulous code walkthrough.

patch authors
=============

Glenn S. Fowler, Google, 2014.

For questions and feedback, use ["gcloud" tag on StackOverflow](http://stackoverflow.com/questions/tagged/gcloud), check out the [Google Cloud SDK's groups page](https://groups.google.com/forum/?fromgroups#!forum/google-cloud-sdk) or send an e-mail directly to google-cloud-sdk@googlegroups.com.

Found a bug? File it in [Cloud SDK issue tracker](https://code.google.com/p/google-cloud-sdk/issues/list).
