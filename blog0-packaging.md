---
layout: default
---

# Packaging Ceph/Mantle Binaries

In this post we will compile Ceph and package it in a Docker image. This makes
it easier to develop and test on a cluster because Docker layering reduces the
time for nodes to pull new binaries. Because Mantle was [merged into
Ceph](https://github.com/ceph/ceph/pull/10887) the build process now includes
Mantle by default!

Tool(s): Docker

## Compiling Ceph and Building an Image

First, get the source code:

```bash
~$ git clone --recursive https://github.com/ceph/ceph.git
```

We will build a Docker image with our custom Ceph binaries. These directions
are adapted from our [Ceph builder
wiki](https://github.com/systemslab/docker-cephdev/wiki/Quickstart). First,
pull in the Ceph image you want to layer your changes onto and tag it so our
Docker containers know where to find it:

```bash
~$ docker pull ceph/daemon:tag-build-master-jewel-ubuntu-14.04
tag-build-master-jewel-ubuntu-14.04: Pulling from ceph/daemon
[... snip ...]
~$ docker tag ceph/daemon:tag-build-master-jewel-ubuntu-14.04 ceph/daemon:latest
~$ docker images
REPOSITORY              TAG                                   IMAGE ID            CREATED             SIZE
ceph/daemon             latest                                e68ed703825f        13 days ago         1.078 GB
```

Now we compile Ceph and build the Docker image with the new binaries:

```bash
~$ wget https://raw.githubusercontent.com/systemslab/docker-cephdev/master/aliases.sh
~$ . aliases.sh
~$ mkdir ceph; cd ceph
~/ceph$ dmake \
          -e GIT_URL="https://github.com/ceph/ceph.git" \
          -e RECONFIGURE="true" \
          -e SHA1_OR_REF="remotes/origin/master" \
          -e CONFIGURE_FLAGS="-DWITH_TESTS=OFF" \
          -e BUILD_THREADS=`grep processor /proc/cpuinfo | wc -l` \
          cephbuilder/ceph:latest \
          build-cmake
```


This tells the builder to pull the source code from `GIT_URL` and checkout
branch `SHA1_OR_REF`. The `RECONFIGURE` flag ensures that the directory is
clean.  `BUILD_THREADS` sets the number of cores to use during the compilation;
in the example above, we use all available cores. `CONFIGURE_FLAGS`
reduces the final size of the image by skipping the insallation of test
binaries. The Ceph source code and compiled binaries end up in the `ceph`
directory.

In the process, the above commands pull the image from DockerHub where we have
an image for building the Ceph master branch (latest) and one for building
Jewel. Check out the DockerHub page
[here](https://hub.docker.com/r/cephbuilder/ceph/tags/) to see which images are
getting pulled. For more information on what is in the image take a look at our
Ceph builder wiki [here](https://github.com/systemslab/docker-cephdev/wiki). 

## Checking the Image:

We should sanity check that image by make sure all the Ceph command line tools
work:

```bash
~/ceph$ docker run --entrypoint=ceph ceph-heads/remotes/origin/master 
[... snip ...]
~/ceph$ docker run --entrypoint=ceph-fuse ceph-heads/remotes/origin/master 
[... snip ...]
~/ceph$ docker run --entrypoint=rados ceph-heads/remotes/origin/master 
rados: symbol lookup error: rados: undefined symbol: _ZN4ceph7logging3Log12create_entryEiiPm
```

If the commands return help menus they are fine; the last error is problematic
and can happen when building images based on the Ceph master branch. Luckily,
this is just a library problem and we can fix it with:

```bash
~/ceph$ docker run \
          --name=fix \
          --entrypoint=/bin/bash \
          -v `pwd`:/ceph \
          ceph-heads/remotes/origin/master \
          -c "cp /ceph/build/lib/libradosstriper* /usr/lib"
~/ceph$ docker commit --change='ENTRYPOINT ["/entrypoint.sh"]' \
          fix ceph-heads/remotes/origin/master
~/ceph$ docker run --entrypoint=rados ceph-heads/remotes/origin/master 
2016-10-31 03:57:02.637232 7fbc821a2a40 -1 did not load config file, using default settings.
rados: you must give an action. Try --help
```

Great. This is what we want. 

## Sharing/Distributing the Image

Now we want to distribute the image on the cluster. We could push it to
DockerHub but this is slow. Instead we set up our own in-house registry that
houses and serves Docker images. For directions for pushing images to DockerHub
and setting up your own registry, see our [Docker Ceph builder
wiki](https://github.com/systemslab/docker-cephdev/wiki/Distributing-Images).
Assuming you have set up your own registry, we can now push to it:

```bash
~/ceph$ docker tag ceph-heads/remotes/origin/master piha.soe.ucsc.edu:5000/ceph/daemon:master
~/ceph$ docker push piha.soe.ucsc.edu:5000/ceph/daemon:master
~/ceph$ curl -X GET http://$REGISTRY_IP:5000/v2/_catalog | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   383  100   383    0     0  15147      0 --:--:-- --:--:-- --:--:-- 15320
{
    "repositories": [
        "ceph/daemon",
    ]
}
```

Now that we have our image in the registry, if we do a Docker pull on each node
we will have an updated image. This works especially well if we are actively
developing because new nodes will only pull binaries that differ from the
binaries in their images; so if you only touched something in the metadata
server only the binaries for the MDS daemon will need to be pulled. More
information on how our builder does this with Docker layers can be found in our
[Docker Ceph builder
wiki](https://github.com/systemslab/docker-cephdev/wiki/Architecture). 
