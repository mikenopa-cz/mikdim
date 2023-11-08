# mikdim - The Mikenopa Docker imager.

Mikdim is a python based tool allowing to build docker images with the codebase of the project itself. When developing a docker based project (like a web application)
you need to maintain at least two runtime environments - development environment (used by project developers) and the production environment (used by people the project
was intended for).

But both environments have different expectations:
* In the development environment we want quick updates. Whenever one changes a file, the container should immediately react. In docker, this is typically done by a mount
  of the application codebase directly into the container.
* In the production environment we don't need these quick updates. But on the other hand, it would be very useful to have just a single production image containing
  even the codebase of the project.

Mikdim is a tool allowing to maintain both types of images as a single Dockerfile. Mikdim distinguishes two types of builds:

1. **development builds** - images to be used for developer's runtime environments (not containing the codebase)
2. **production builds** - images to be used in production runtime environments (including the codebase)

However `mikdim` supports development builds, there is nothing interesting on development builds. Development builds may be fully covered by regular docker building
tools and using `mikdim` is redundant. Development builds are supported only for conventional reasons so that both development and production builds may be done in
a similar way.

## Installation

`mikdim` is just a single file binary being run in python. It should be sufficient to copy the binary into a directory being in your PATH environment variable.
If you are using php and composer, you may install mikdim using composer by requiring the package `mikenopa/mikdim`:

```bash
composer require mikenopa/mikdim
```

If installed using composer, it is not clear, if mikdim should be installed as a dev depencency or a regular dependency. It depends on the docker image creation process
being set up.

If installed using composer, you may run mikdim using:
```bash
vendor/bin/mikdim
```

## Quick start

* put a `mikdim.yaml` config file into your project. (See: [mikdim.example.yaml](mikdim.example.yaml))
* run `mikdim` in your project root
* your production docker images are built and tagged properly


## Build procedures detailed

### Development build

* Run the command specified as `prepare-command-dev` in the image configuration (usually there is no reason to use such a command, but it is possible)
* Build the image according to the specified Dockerfile with the environment variables specified as `environment-dev` in the image configuration
* Tag the resulting image (see later)

### Production build

* Run the command specified as `prepare-command` in the image configuration. Such a command may be used to prepare the codebase for putting it into the container.
* Build the image according to the specified Dockerfile with the environment variables specified as `environment` in the image configuration
* Put the codebase inside of the image as a next building step (in fact implemented as a standalone docker file).
* Invoke the command specified as `postprocess` in the image configuration allowing to change the image itself after it is copied into the image
* Tag the resulting image (see later)

### Tagging images

Each image needs to specify a name in the image confiuration. This is always used as the base for the docker image name and cannot be changed by any options. But there are
available multiple ways, how to control the version part of the image tag. (i.e. the part after the colon `:`).

This holds for production builds:
* Production builds are tagged by the semantic version number parsed from the git tag. You may specify the version manually by `--project-version <version>` if you don't use git.
* Production builds are tagged by the branch name of the current branch if the current branch is on the list of allowed branches. `production`, `staging`, `main` and `master` is the
  default list of allowed branches. You may specify the branch name manually by `--project-branch <branch>` if you don't use git.
* If `--latest` flag is used, the production build is also tagged by the `latest` tag.
* You may specify any other tag by passing the option `--tag <tag>`.
* You may completely disable the version tagging by specifying `--no-project-version`
* You may completely disable the branch tagging by specifying `--no-project-branch`
* You may change the list of allowed branches by `--allow-branch <branch>`. Once one branch is allowed by this option, the default allowed branches will not be vaild anymore.
* You may also allow all branches to be used as tags by specifying `--allow-all-branches`

This holds for the development builds:
* Development builds are tagged by any tag specified using `--tag <tag>` option.
* If no tag is specified, the default `dev` tag is used.

When the production build is created, it is necessary to build a semiproduct image (similar to development build, but with production environment variables). This semiproduct image
is used only temporarily and is automatically cleaned up after the production build is finished. But this semiproduct image needs to be tagged as well. For this purpose the default
tag `semi-tmp` is used, but this default tag may be changed by the option `--semi-suffix <semiproduct-tag>`.

## The `mikdim` command

Type

```bash
mikdim --help
```

for a detailed help of all supported options. By default, `mikdim` searches for the `mikdim.yaml` file in the current working directory. But you may specify another file by the option
`--project <dir-or-file>`. Directories or files may be specified as projects. If a directory is specified, mikdim will search for `mikdim.yaml` inside of that directory. If a file is
specified, mikdim will use exactly this file as the project file.

Supported options:

* `--dev` (`-d) - invoke a development build instead of a production build
* `--latest` (`-l`) - tag the production build with the version tag `latest`
* `--tag <tag>` (`-t`) - tag the build with an arbitrary version tag
* `--project <project>` (`-p`) - specify some another project file (or directory, see above)
* `--project-version <version>` (`-V`) - specify a custom project version instead of taking it from git tag
* `--project-branch <branch>` (`-B`) - specify a custom project branch instead of taking it from git
* `--no-project-version` - disable production build version tagging
* `--no-project-branch` - disable production build branch tagging
* `--preserve-environment` (`-e`) - use environment variables passed to the commands. Without that option, only environment variables from `mikdim.yaml` are passed to the build process
* `--allow-branch <branch>` (`-a`) - specify a branch name, where branch tagging is allowed. If the branch is not on the allow list, branch tag will not be used
* `--allow-all-branches` (`-A`) - allow all branches to be uased as branch tags
* `--image <image-name>` (`-i`) - build only the specified image from all images being specified in `mikdim.yaml`
* `--semi-suffix <tag>` (`-s`) - specify a custom semiproduct tag (temporary tag, it is removed after production build is completed).


## The mikdim.yaml configuration file

See the example file [mikdim.example.yaml](mikdim.example.yaml). The `mikdim.yaml` file is a manifest file containing all necessary information for building an image. Each image
allows these configuration variables:
* **name** - the name of the image being built [required]
* **build** - path to the docker context used to build the image (i.e. Dockerfile path) [required]
* **project-dir** - the directory inside of the docker image, where the codebase should be copied to
* **postprocess** - a command to be invoked inside of the image after the codebase is put on the proper place
* **prepare-command** - a command to be invoked before the production building process is started (useful for preparing the codebase to the production state)
* **prepare-command-dev** - a command to be invoked before the development building process is started (not very useful)
* **subdir** - specify a subdirectory being used as the codebase
* **exclude** - exclude specific files or directories from the codebase (specified relatively)
* **environment* - a list of environment variables being passed to the image production building process
* **environment-dev* - a list of environment variables being passed to the image development building process

Example configuration file:
```yaml
images:
    - name: mikm-test-image
      build: "docker/test-image"
      project-dir: "/var/www/html"
      prepare-command: "bin/prepare"
      exclude: ["docker"]
      environment:
        BUILD_ENV: production
```
