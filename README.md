# Bioweb Docker images

Docker images for [bioweb](http://bioweb.sourceforge.net/en/index.html), the framework to build genetic data analysis software.

## Usage

Depending on your configuration, you may have to prepend the `docker` command with `sudo`.

### Try the example application

```bash
# If you want the container to persist after exiting, remove the --rm option.
docker run -i -p 9000:9000/tcp -t --rm krystiancha/bioweb r=l
```

Then, browse to: http://localhost:9000

### Print available options

```bash
# If you want the container to persist after exiting, remove the --rm option.
docker run -i -p 9000:9000/tcp -t --rm krystiancha/bioweb
# or
docker run -i -p 9000:9000/tcp -t --rm krystiancha/bioweb -h
```

### Try from a shell

```bash
# If you want the container to persist after exiting, remove the --rm option.
docker run -i -p 9000:9000/tcp -t --entrypoint /bin/bash --rm krystiancha/bioweb
```

Then, in the container:

```bash
scons r=l
```

Then, browse to: http://localhost:9000

### Development images

This image is build on the `bioweb` image. It includes `nano`, `vim` and `emacs`.

```bash
# If you want the container to persist after exiting, remove the --rm option.
docker run -i -p 9000:9000/tcp -t --entrypoint /bin/bash --rm krystiancha/bioweb:dev
```

## Building

During the buid process Boost is built. You can decide how many threads will the compiler use by setting `boost_build_threads` as can be seen below. `6` or `7` should be safe value for a system with 8 G of memory. If you tag the image `bioweb` instead of `krystiancha/bioweb` (as shown below), remember to refer to it as `bioweb` when using the image.

```bash
git clone https://git.sr.ht/~krystiancha/bioweb-docker
cd bioweb-docker

# bioweb
docker build -t bioweb --build-arg boost_build_threads=7 bioweb

# bioweb:dev
docker build -t bioweb:dev bioweb-dev
```

## Known problems

- [ ] `scons r=f` doesn't work due to lack of Chrome in the image.
