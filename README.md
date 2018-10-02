Buildfarm Dockerfiles
===========

This is a collection of Dockerfiles to create containers to run the PostgreSQL
Buildfarm client. They need to run in Docker >= 18.03 due to a prior bug
in setting ownership of mounted directories.

Build the containers in the usual way. It's useful to pass in your uid/gid
so the user writing to the buildroot will have the right privileges:

```
docker build --rm=true -t my-buildfarm-client -f Dockerfile.foo \
	   --build-arg uid=`id -u` --build-arg gid=`id-g` .
```

The client needs to be attached to some persistent storage, which will contain
all the state for the buildfarm client, as well as the ephemeral build
artefacts. You will also need to get a config file. First do this:

```
mkdir buildroot
docker run --rm=true -v `pwd`/buildroot:/app/buildroot my-buildfarm-client \
	   cp build-farm.conf.sample buildroot/build-farm.conf
```

Now edit buildroot/build-farm.conf:

* change the animal name
* change the buildroot to "/app/buildroot"
* turn off git mirror
* turn on vpath builds

These last two are not required but are recommended.

Now you can run the client:

```
docker run --rm=true -v `pwd`/buildroot:/app/buildroot my-buildfarm-client \
	   run_build.pl --config=buildroot/build-farm.conf --test
```

On Alpine you need to delay the check step until after the install, so
add this to the above command

````
--delay-check
````

Alpine also has a few other things that don't seem to work, such as nls, ldap,
and gssapi.

If you want to do a `--from-source` build, then mount the source directory
as well as the buildroot. There is a supplied config file for these builds,
which expects the source to be mounted at `/app/pgsrc`:

````
docker run --rm=true -v `pwd`/buildroot:/app/buildroot \
	   -v '/path/to/src:/app/pgsrc' \
	   my-buildfarm-client \
	   run_build.pl --config=build-fromsource.conf
````
