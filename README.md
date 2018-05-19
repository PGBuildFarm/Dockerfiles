Dockerfiles
===========

This is a collection of Dockerfiles to create containers to run the PostgreSQL
Buildfarm client. They need to run in Docker >= 18.03 due to a prior bug
in setting ownership of mounted directories.

Build the containers in the usual way:

```
docker build -t my-buildfarm-client -f Dockerfile.foo context
```

The client needs to be attached to some persistent storage, which will contain
all the state for the buildfarm client, as well as the ephemeral build
artefacts. You will also need to get a config file. First do this:

```
mkdir buildroot
docker run -v `pwd`/buildroot:/app/buildroot my-buildfarm-client \
	   cp buildfarm.conf.sample buildroot/build-farm.conf
```

Now edit buildroot/build-farm.conf:

* change the animal name
* change the buildroot to "/app/buildroot"
* turn off git mirror
* turn on vpath builds

These last two are not required but are recommended.

Now you can run the client:

```
docker run -v `pwd`/buildroot:/app/buildroot my-buildfarm-client \
	   ./run_build.pl --config=buildroot/build-farm.conf --test
```


