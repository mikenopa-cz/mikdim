# The mikdim/mikdeb building suite

This package contains two building tools:

* [mikdim](README.mikdim.md) is a tool for building docker images
* [mikdeb](README.mikdeb.md) is a tool for building debian packages

Both tools are implemented in a single python-based binary. The binary may be just symlinked to a different name to run the secnod tool. This logic holds:

* if the target binary name or target symlink name is `mikdeb`, the mikdeb tool is invoked
* mikdim is invoked otherwise

Both tools internally use docker for building. While mikdim uses docker for building docker images, mikdeb uses a docker container for invoking the privileged
part of creating the debian package. (file/dir owners and permissions needs to be set properly)

## Installation

The mikdim/mikdeb is just a single binary which does not need any special installation procedure. These packages needs to be installed to run mikdim/mikdeb properly:

* python3
* python3-yaml
* python3-schema
* docker.io

If you want to make a regular installation using a deb file, you may build it using mikdeb itself. Installation procedure:

1. install the dependencies mentioned above
2. clone the mikdim repository to your local computer
3. run `./mikdeb --git` in the root of the repository
4. install the created debian package by `apt install ./mikdim_*.deb`
5. mikdim and mikdeb are now system-wide commands


## Composer package

However mikdim/mikdeb is a python based project, we use it internally to deploy mainly php-based project. Therefore we made mikdim/mikdeb available also via composer:

```bash
composer require mikenopa/mikdim
```

## Known issues

* Documentation needs to be extended.
* Manual pages should be written.

