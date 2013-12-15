cpm-bson
========

[![Build Status](https://travis-ci.org/iauns/cpm-libuv.png)](https://travis-ci.org/iauns/cpm-libuv)

CPM libuv external.

Usage
-----

```c++
#include <uv.h>
```

See libuv's github page: https://github.com/joyent/libuv.git .

Platform Issues
===============

Mac OS X
--------

Builds, but `fs_event_watch_dir` is not supported.

Windows
-------

Windows is supported by libuv, but not currently implemented in this CPM
external. Stay tuned.

