# mikdim - The Mikenopa Docker imager.

Mikdim is a python based tool allowing to build docker images with the codebase of the project itself. When developing a docker based project (like a web application)
you need to maintain at least two runtime environments - development environment (used by project developers) and the production environment (used by people the project
was intended for).

But both environments have different expectations:
* In the development environment we want quick updates. Whenever one changes a file, the container should immediately react. In docker, this is typically done by a mount
  of the application codebase directly into the container.
* In the production environment we don't need these quick updates. But on the other hand, it would be very useful to have just a single production image containing
  even the codebase of the project.

Mikdim is a tool allowing to maintain both types of images as a single Dockerfile. The intended usage is as follows:

* If you want to build a development image, you will just use regular `docker build` and `mikdim` itself is not necessary.
* If you want to build a production image, you will set up a `mikdim.yaml` file, which describes the process how to finalize the production image from the development image.
  The main goal of the finalization is to add the codebase of the project itself. `mikdim.yaml` contains instructions for mikdim, how the codebase should be added into the final image.

## Quick start

* put a `mikdim.yaml` config file into your project. (See: [mikdim.example.yaml](mikdim.example.yaml))
* run `mikdim` in your project root
* your docker images are built
