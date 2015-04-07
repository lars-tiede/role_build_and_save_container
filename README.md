## About this role

This role rsyncs a directory with a Dockerfile in it to a "build" host, and (on change) builds a docker image there and saves and downloads that image to the ansible control host. The image can then be loaded into Docker daemons anywhere. [This](https://github.com/lars-tiede/role_install_container_from_archive) role can do this for you.

You must have root access to the build host, and docker must be installed on the build host. It must also have a user with a home directory that can be used for syncing directories and making archives (more on that below).

rsync is called with the -FF option, so that .rsync-filter files with filter rules are respected, and the rules files themselves not synced. This makes it easy in your Dockerfile to not copy unneccessary data into the container when using ADD or COPY directives - just sync the files you need in the container.

You can specify additional options for rsync in an optional role variable. A useful application for this is to specify more rsync filter rules that only apply when rsyncing your project directory for the purpose of building a container (see example below).


### Role variables

The role uses the following _mandatory_ variables:

* *host_build_user*: user on build host with a home directory that we can rsync directories to and store image archives in. This can be any existing user, but it is recommended to have a dedicated user account with home directory on the build host for that, in order to keep all the pollution in one place and make sure that there are no conflicts. Example: "ansible-docker-build"
* *dockerfile_dir*: directory with Dockerfile describing the container you want
  to build, relative from the playbook's directory. As per Docker convention, this directory probably contains everything that will end up in the container, and then some (which you can filter using rsync filters). Trailing slash is optional.
* *image_name*: Docker name of the image to build. Example: "lars.tiede/incoming"
* *image_tag*: Docker tag of the image to build. Example: "1.0.2"
* *image_file_name*: name (before .tar.gz) the image archive file should get. Example: "incoming"
* *image_storage_dir*: path to the local directoory (on the ansible control host) in which to store the image archive file, relative to the playbook's directory. The directory must exist. Example: "docker\_images"


The following variables are _optional_:

* *extra_rsync_opts*: specify extra options for rsync, like so: '--opt1=blah,--opt2=blah'. The default value is ''. Example: *'--filter="dir-merge /.rsync-filter-docker",--filter="- .rsync-filter-docker"'*. This would allow you to have specific rsync filter rules in .rsync-filter-docker files in any synced directory.
* *remove_image_from_host*: if set to True (which is the default), both the image and the image archive will be removed from the build host after build. Set to False to retain both. Removing images will remove all images 'between' the base image you built from and your image, so that subsequent builds will not be able to use the image cache to speed up building. You probably want this behavior in production, but when testing, being able to use the image cache might be beneficial.


## How to install

This role has a dependency to another role which you can get [here](https://github.com/lars-tiede/role_has_docker-py). You install that role in exactly the same way as this one. Below, you find the installation procedure for both roles.

Clone the repositories in your roles directory into appropriately named directories.

```bash
you@your-role-dir$ git clone https://github.com/lars-tiede/role_has_docker-py.git has_docker-py
you@your-role-dir$ git clone https://github.com/lars-tiede/role_build_and_save_container.git build_and_save_container
```

If your roles directory is itself in a git repository, you can't do the above. You have two options (that I know of) then, which are outlined [here](https://github.com/lars-tiede/role_has_docker-py/blob/master/README.md#include-in-another-git-repository). Just replace the repository and role name in the instructions. For the impatient, here is how you include the roles into your git repository as submodules (which is one of several ways you can do this):

```bash
you@your-project-dir$ git submodule add https://github.com/lars-tiede/role_has_docker-py.git roles/has_docker-py
you@your-project-dir$ git submodule add https://github.com/lars-tiede/role_build_and_save_container.git roles/build_and_save_container
```
