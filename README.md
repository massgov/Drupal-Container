Drupal Application Container
============================

These [Dockerfiles](https://docs.docker.com/engine/reference/builder/) define the [massgov/drupal-container](https://hub.docker.com/r/massgov/drupal-container) images.  These images are automatically built by Docker Hub.

[![Docker Pulls](https://img.shields.io/docker/pulls/massgov/drupal-container.svg)](https://hub.docker.com/r/massgov/drupal-container)
[![Docker Stars](https://img.shields.io/docker/stars/massgov/drupal-container.svg)](https://hub.docker.com/r/massgov/drupal-container)


Variants
--------
The variants of this image are:
  [`ci`](./ci/) - An image appropriate for use in continuous integration environments.  Closely mirrors Acquia's application servers, except that mail is disabled.
  [`dev`](./dev/) - An image appropriate for use in development environments.  Adds Xdebug, Blackfire, ...

Environment variables
------------------
1. XDEBUG_ENABLE: Adds xdebug extension which is configured to autostart.
2. MEMORY_LIMIT: Set the PHP memory limit that is used.
3. XDEBUG_REMOTE_HOST: Defaults to `host.docker.internal` (good for OSX). If using Linux, see https://github.com/docker/for-linux/issues/264 and https://biancatamayo.me/blog/2017/11/03/docker-add-host-ip/
4. XDEBUG_IDEKEY: Defaults to `PHPSTORM`.

Making changes
--------------

1. Make changes to the Dockerfile for the desired variant (2022 update: we only use ci variant)
2. Test changes by building the image locally using `docker-compose build`
3. Note changes in [CHANGELOG.txt](./CHANGELOG.txt).
4. Commit and push changes to this repository: 
    ```
    git commit -m "My message"
    git push origin master
    ```
5. Optionally, tag a new release:
    ```bash
    git tag 1.0.0
    git push origin 1.0.0
    ```
    When you tag a release, you should also update your code to use the new version as soon as it's available.  You can check on the availability on the [build details](https://hub.docker.com/r/massgov/drupal-container/builds/) page.

Changes that are pushed to the `master` branch are built automatically by Docker hub.  For example, a change pushed to master affecting the `ci` variant would update `massgov/drupal-container:ci`.  Tags are also built automatically and create a new tagged version of the image.  For example, a new tag `4.1.0` would create new versions of all variants, which would be available using `massgov/drupal-container:4.1.0-ci` etc.
