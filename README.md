TYPO3 Flow in Docker
====================

This repository contains a Docker base image for building containerized
**TYPO3 Flow** applications.

Use for development
-------------------

While in development, you may want to mount your TYPO3 Flow application from a
host-local directory into your container.

To do this, start your container like this:

    docker run --name flow-db \
               -e MYSQL_ROOT_PASSWORD=secret \
               -e MYSQL_PASSWORD=othersecret \
               -e MYSQL_USER=flow \
               -e MYSQL_DATABASE=flow \
               mariadb:latest
    docker run --name flow-web \
               --link flow-db:db \
               -e FLOW_DB_HOST=db \
               -e FLOW_DB_USER=flow \
               -e FLOW_DB_PASSWORD=othersecret \
               -v $PWD:/var/www \
               -p 80:80 \
               martinhelmich/flow:dev

The `-v $PWD:/var/www` mounts your local working directory into the container's
`/var/www` directory, from which the Flow application is served. The `-p 80:80`
maps the container port 80 to your host port 80, allowing your TYPO3 Flow
application to be accessed by `http://localhost`.

Use for production
------------------

In production, you'd typically want to bake your TYPO3 Flow application into a
custom Docker image. This is possible with this image, too, due to an
`ONBUILD ADD . /var/www` instruction in the Dockerfile.

Simply include a custom `Dockerfile` in your Flow application's root directory
with the following contents:

    FROM martinhelmich/flow:prod

Then build the image:

    docker build -t you/custom-flow-app .

And run it:

    docker run --name flow-web \
               --link flow-db:db \
               -p 80:80 \
               you/custom-flow-app

The `prod` image differs from the `dev` image in a few aspects:

1. Of course, the `FLOW_CONTEXT` environment variable is set to `PRODUCTION` in
   the `prod` image. Duh.
2. On startup, the TYPO3 Flow Cache will be warmed up by executing a
   `./flow flow:cache:warmup` on startup.
3. Furthermore, Doctrine migration will be executed if necessary on startup
   (unless the `FLOW_DB_DISABLE` environment variable was set).

Environment variables
---------------------

You can configure this container image's behaviour using the following environment variables (which you can specify at container startup with a `-e <key>=<value>` flag):

1. **FLOW_DB_USER**. Database user to connect with.
2. **FLOW_DB_HOST**. The database host.
3. **FLOW_DB_PASSWORD**. The database user's password.
4. **FLOW_DB_DISABLE**. Set this if your Flow application does not use a database.