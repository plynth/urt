urt
===

Utility Runner Tool

(Name to be decided. I want something short, unique, and single syllable. 
I want to avoid "docker" in the name because it can also work with OCI)

``urt`` is a tool to run command line utility Docker and OCI images. It
makes CLI tools simple to package, distribute, and use. It is particularly
useful for sharing development tools with your team.

The images are ran in containers with the caller's working directory,
environment, and user::

    $ whoami
    admin
    $ urt whoami
    admin
    $ cd $HOME
    $ urt pwd
    /home/admin
    $ export FOO=1
    $ urt env
    FOO=1


## Running an application

Download the image::

    # urt --pull <imagename>:<tag>
    $ urt --pull urt/hello-world:latest

Now commands can be ran::

    $ urt hello-world
    Hello!

By default, commands are installed as the base image name. If there
is more than one image with the same basename, the first image added
will be used. the full name can be specified to use the other image::

    $ urt --pull dockercloud/hello-world:latest
    $ urt dockercloud/hello-world
    Hello world!


For images on Docker hub that do not have a namespace, use `_`::

    $ urt --pull hello-world:latest
    $ urt _/hello-world
    Hello from Docker!

## Versioning and Tags

By default ``urt`` will look run the first found tag of:

- ``urt`` -- This tag can be used to set which version of an image
    will be ran by default.
- ``latest`` -- Most images provde a ``latest``
- The only tag installed

To run a specific tag, specify the full image name and tag:


    $ urt --pull urt/hello-world:2.0
    $ urt urt/hello-world:2.0
    Hello 2.0!

### Updating

To pull the latest of all images with the ``latest`` tag::

    $ urt --upgrade

    
## Invoking commands without ``urt``

### Scripts

Wrapper scripts can be added for each image::

    $ urt --install urt/hello-world

This will place an executable in ``/usr/local/urt/bin``. Add this
directory to your path and you will then be able to execute
commands without the ``urt`` command::
    
    $ hello-world
    Hello!

You can change the name of the script::

    $ urt --install urt/hello-world=greet
    $ greet
    Hello!

Normally, scripts use the default versions. You can override the 
version to script::

    $ urt --alias urt/hello-world:2.0=greet2
    $ greet2
    Hello 2.0!


### Aliases

Aliases can be used so you don't have to specify the ``urt`` command
for an installed application.

To add an alias::

    $ urt --alias urt/hello-world

The ``--alias`` flag will create an alias in your shell. To create aliases
automatically for new shells, first add the alias with ``urt --alias <image>``
then add to your shell profile::

    eval $(urt --aliases)

You can change the name of the alias::

    $ urt --alias urt/hello-world=greet
    $ greet
    Hello!

Normally, aliases use the default versions. You can override the 
version to alias::

    $ urt --alias urt/hello-world:2.0=greet2
    $ greet2
    Hello 2.0!


## Directories

By default, ``urt`` only mounts the current working directory. To have
``urt`` mount other directories, add the following to ``~/.urt/config``::

    directories=[
        "/home/admin"
    ]

The directories will be mounted from and to the same place in the container.
This is so when you run commands and get errors back, the path name will
match the host's directory name.


## Sharing with a Urtfile

A project can include a ``Urtfile`` that describes all
the utilities the team uses for the project::

    $ cat Urtfile    
    images = [
        # Image with an alternate name
        hello-world:latest=greet,
        urt/hello-world:2.0,
    ]

To pull all the images specified in ``Urtfile``::

    $ urt --pull

## Directories

To mount the project directory when running commands in the ``Urtfile`` 
directory, add it to the ``Urtfile``::

    directories=[
        "${URTFILE_DIR}"
    ]


### Urtfile Scripts

To script all the images specified in ``Urtfile``::

    $ urt --install

This will install the scripts in ``${URTFILE_DIR}/.urt/bin`` so
it does not conflict with your system files. You can add
``${URTFILE_DIR}/.urt/bin`` to your path in the shell by running::

    $ PATH=$(urt --path)

The changes will not persist. To have ``urt`` automatically update the
path when you change to the ``Urtfile`` directory or one of its children, 
add the following to your shell::

    (TODO whatever magic makes this work)

### Aliases

To alias all the images specified in ``Urtfile``::

    $ urt --alias

The changes will not persist. To have ``urt`` automatically update the
aliases when you change to the ``Urtfile`` directory or one of its children, 
add the following to your shell::

    (TODO whatever magic makes this work)

## Making images for urt

Practically any Docker or OCI image is automatically compatible with ``urt``.

### Urt command

By default, ``urt`` will run the entrypoint or command of an image. 
However, the author of an image may not want to ``urt`` to use that command. 
For example, the image may normally run a webserver, but ``urt`` should
run debugger. To change the urt command, add a ``LABEL`` to the image::

    LABEL=io.urtcli.command=/bin/debug

``urt`` will now run this command instead of the default entrypoint or command.

### Open Ports

If your ``urt`` command runs some sort of server, you probably want ports 
opened on the host::

    LABEL=io.urtcli.ports=[8080]



