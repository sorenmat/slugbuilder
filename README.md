# (Heroku-ish) Slug Builder
A tool using [Docker](http://docker.io) and [Buildpacks](https://devcenter.heroku.com/articles/buildpacks) to produce a Heroku-like [slug](https://devcenter.heroku.com/articles/slug-compiler) given some application source.

## What does it do exactly?

It's a Docker container that takes an uncompressed tarball of an application source piped to it. The source is run through buildpacks, then if it's detected as a supported app it will be compiled into a gzipped tarball ready to be run somewhere. 

## Using Slug Builder

First, you need Docker. Then you can either pull the image from the public index:

	$ docker pull flynn/slugbuilder

Or you can build from this source:

	$ cd slugbuilder
	$ make

When you run the container, it always expects a tar of your app source to be passed via stdin. So let's run it from a git repo and use `git archive` to produce a tar:

	$ id=$(git archive master | docker run -i -a stdin flynn/slugbuilder)
	$ docker wait $id
	$ docker cp $id:/tmp/slug.tgz .

We run slugbuilder, wait for it to finish using the id it gave us, then copies out the slug artifact into the current directory. If we attached to the container with `docker attach` we could also see the build output as you would with Heroku. We can also *just* see the build output by running it with stdout:

	$ git archive master | docker run -i -a stdin -a stdout flynn/slugbuilder

We still have to look up the id and copy the slug out of the container, but there's an easier way!

	$ git archive master | docker run -i -a stdin -a stdout flynn/slugbuilder - > myslug.tgz

By running with the `-` argument, it will send all build output to stderr (which we didn't attach here) and then spit out the slug to stdout, which as you can see we can easily redirect into a file.

Lastly, you can also have it PUT the slug somewhere via HTTP if you give it a URL as an argument. This lets us specify a place to put it *and* get the build output via stdout:

	$ git archive master | docker run -i -a stdin -a stdout flynn/slugbuilder http://fileserver/path/for/myslug.tgz

## Caching

To speed up slug building, it's best to mount a volume specific to your app at `/tmp/cache`. For example, if you wanted to keep the cache for this app on your host at `/tmp/app-cache`, you'd mount a read-write volume by running docker with this added `-v /tmp/app-cache:/tmp/cache:rw` option:

	docker run -v /tmp/app-cache:/tmp/cache:rw -i -a stdin -a stdout flynn/slugbuilder


## Buildpacks

As you can see, slugbuilder supports a number of official and third-party Heroku buildpacks. You can change the buildpacks.txt file and rebuild the container to create a version that supports more/less buildpacks than we do here. You can also bind mount your own directory of buildpacks if you'd like:

	docker run -v /my/buildpacks:/tmp/buildpacks:ro -i -a stdin -a stdout flynn/slugbuilder

## Base Environment

The Docker image here is based on [cedarish](https://github.com/progrium/cedarish), an image that emulates the Heroku Cedar stack environment. All buildpacks should have everything they need to run in this environment, but if something is missing it should be added upstream to cedarish.

## License

BSD

## Flynn 

[Flynn](https://flynn.io) is a modular, open source Platform as a Service (PaaS). 

If you're new to Flynn, start [here](https://github.com/flynn/flynn).

### Status

Flynn is in active development and **currently unsuitable for production** use. 

Users are encouraged to experiment with Flynn but should assume there are stability, security, and performance weaknesses throughout the project. This warning will be removed when Flynn is ready for production use.

Please report bugs as issues on the appropriate repository. If you have a general question or don't know which repo to use, report them [here](https://github.com/flynn/flynn/issues).

## Contributing

We welcome and encourage community contributions to Flynn.

Since the project is still unstable, there are specific priorities for development. Pull requests that do not address these priorities will not be accepted until Flynn is production ready.

Please familiarize yourself with the [Contribution Guidelines](https://flynn.io/docs/contributing) and [Project Roadmap](https://flynn.io/docs/roadmap) before contributing.

There are many ways to help Flynn besides contributing code:

 - Fix bugs or file issues
 - Improve the [documentation](https://github.com/flynn/flynn.io) including this website
 - [Contribute](https://flynn.io/#sponsor) financially to support core development

Flynn is a [trademark](https://flynn.io/docs/trademark-guidelines) of Apollic Software, LLC.
