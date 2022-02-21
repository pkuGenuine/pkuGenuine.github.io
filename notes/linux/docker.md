# Docker

## File Permissions

Docker host shares file permissions with containers (at least, in Linux). Whenever we create a file on host using a user with UID x, this files will have x as owner UID inside the container, too. And this will happen no matter if there is a user with UID equal to x inside the container. 

> Aside:
> 
> Since there is only one kernel, there is also only one set of UIDs and GIDs for the host and all the containers.
> However, user names and group names are not shared! They are not part of the kernel. They are managed by external (to the kernel) tools ( like `/etc/passwd` ) who map user names to UID and group names to GIDs. 


### Solutions for Production environment

Starting from Dockerfile, we can change the user under which the container ( and so, the image build process ) is executed by using the “USER” directive. So, if we have an Apache container:

~~~Dockerfile
FROM ubuntu:16.04
# switch to a non-privileged user
USER apache
~~~

This will also affect all `RUN`, `CMD` and `ENTRYPOINT` directives located after the `USER` directive. Remember to create the user, if not already exists:

~~~Dockerfile
RUN groupadd -r apache && adduser apache
~~~

Some services create a user during their installation, so we don’t have to do it ourselves.

As you may see, in case code is deployed through a `COPY` command. the solution is already there as long as we place the `USER` directive before the `COPY`. Otherwise, we can just use a `RUN` directive and changes the ownership of the application files manually.

~~~Dockerfile
ARG user=jenkins
RUN chown -R ${user} "$JENKINS_HOME" /usr/share/jenkins/ref
~~~

> Aside:
> 
> `ARG` is only available during the build of a Docker image ( `RUN` etc ), not after the image is created and containers are started from it ( `ENTRYPOINT`, `CMD` ).

### Solutions for Local Development Environment

All solutions boil down to the same concept. Changing user UID to match the owner of the files, or change the file ownership to match the user’s UID. However, setting up a development environment is a bit more complicated than the case of a production system. Do not be tricked!