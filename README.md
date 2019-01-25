# docker-volume-gluster

     No problem for the confusion, the docker docs doesn't help on this subject and it is quite confusing. Basically, managed plugin are legacy plugin inside a container that can be distributed as an image via registry.
     To build the managed plugin use: make docker-plugin (https://github.com/sapk/docker-volume-gluster/blob/old/Makefile#L42)
     This will build the image.
     You can enable it with make docker-plugin-enable (https://github.com/sapk/docker-volume-gluster/blob/old/Makefile#L68)

     If you want to publish the result to a registry you can set custom env variable

     PLUGIN_USER ?= sapk
     PLUGIN_NAME ?= plugin-gluster
     PLUGIN_TAG ?= latest
     and use the command docker-plugin-push to publish to docker hub.

     Ex:

     #To build lukicsl/plugin-gluster:armv7l
     PLUGIN_USER=lukicsl PLUGIN_TAG=armv7l make docker-plugin
     #To test lukicsl/plugin-gluster:armv7l
     PLUGIN_USER=lukicsl PLUGIN_TAG=armv7l make docker-plugin-enable
     #To publish to docker hub lukicsl/plugin-gluster:armv7l
     PLUGIN_USER=lukicsl PLUGIN_TAG=armv7l make docker-plugin-push
     Please let me know if it works (normally it should but I never try).
     I could try to setup multi-arch plugin like I have done for some of my other docker images.

Status : **proof of concept (working)**

Use GlusterFS cli in the plugin container so it depend on fuse on the host.

## Docker plugin (New & Easy method) [![Docker Pulls](https://img.shields.io/docker/pulls/sapk/plugin-gluster.svg)](https://hub.docker.com/r/sapk/plugin-gluster) [![ImageLayers Size](https://img.shields.io/imagelayers/image-size/sapk/plugin-gluster/latest.svg)](https://hub.docker.com/r/sapk/plugin-gluster)
```
docker plugin install sapk/plugin-gluster
docker volume create --driver sapk/plugin-gluster --opt voluri="<volumeserver>:<volumename>" --name test
docker run -v test:/mnt --rm -ti ubuntu
```

## Create and Mount volume
```
docker volume create --driver sapk/plugin-gluster --opt voluri="<volumeserver>,<otherserver>,<otheroptionalserver>:<volumename>" --name test
docker run -v test:/mnt --rm -ti ubuntu
```

## Docker-compose
```
volumes:
  some_vol:
    driver: sapk/plugin-gluster
    driver_opts:
      voluri: "<volumeserver>:<volumename>"
```


## Additionnal docker-plugin config
```
docker plugin disable sapk/plugin-gluster

docker plugin set sapk/plugin-gluster DEBUG=1 #Activate --verbose
docker plugin set sapk/plugin-gluster MOUNT_UNIQ=1 #Activate --mount-uniq

docker plugin enable sapk/plugin-gluster
```



## Legacy plugin installation
For Docker version 1.12 or below, the managed plugin system is not supported. This also happens if the plugin is not installed via
`docker plugin install`.
[Docker's new plugin system](https://docs.docker.com/engine/extend/) is the preferred way to add drivers and plugins, where the plugin is just
an image downloaded from registry containing the executable and needed configuration files. You can run both legacy and new plugins
in Docker versions above 1.12, but be aware that legacy plugins will not show up on `docker plugin ls`. They will be listed instead under `plugins` on `docker info`.

That way, the driver's name will be just `gluster` (in both the CLI and Compose environments):

#### Build
```
make
```

#### Start daemon
```
./docker-volume-gluster daemon
OR in a docker container
docker run -d --device=/dev/fuse:/dev/fuse --cap-add=SYS_ADMIN --cap-add=MKNOD  -v /run/docker/plugins:/run/docker/plugins -v /var/lib/docker-volumes/gluster:/var/lib/docker-volumes/gluster:shared sapk/docker-volume-gluster
```

For more advance params : ```./docker-volume-gluster --help OR ./docker-volume-gluster daemon --help```
```
Run listening volume drive deamon to listen for mount request

Usage:
  docker-volume-gluster daemon [flags]

Flags:
  -h, --help         help for daemon
      --mount-uniq   Set mountpoint based on definition and not the name of volume

Global Flags:
  -b, --basedir string   Mounted volume base directory (default "/var/lib/docker-volumes/gluster")
  -v, --verbose          Turns on verbose logging
```

#### Create and Mount volume
```
docker volume create --driver gluster --opt voluri="<volumeserver>:<volumename>" --name test
docker run -v test:/mnt --rm -ti ubuntu
```



## Performances : 
As tested [here](https://github.com/sapk/docker-volume-gluster/issues/10#issuecomment-350126471), this plugin provide same performances as a gluster volume mounted on host via docker bind mount.

## Inspired from :
 - https://github.com/ContainX/docker-volume-netshare/
 - https://github.com/vieux/docker-volume-sshfs/
 - https://github.com/sapk/docker-volume-gvfs
 - https://github.com/calavera/docker-volume-glusterfs
 - https://github.com/codedellemc/rexray
