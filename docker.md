# Docker
## Basic commands

* `docker ps` - list running containers. add `-a` to list all containers.
* `docker images` - list docker images. all `-a` to list intermediate images
* `docker run [image]` - start a docker container and run the [image] in the container
  * `-p host:container` - port forwarding between the host and container
  * `-v /absolute/path/to/local/dir:/absolute/path/on/image` - setup a volume in the container to listen for changes in the host filesystem 
* `docker image remove [image]` - remove image

[DockerHub](http://hub.docker.com) can be used to find images.


### Building your own docker image
#### Provisioning your image
A Dockerfile is used to configure the images, images themselves are used to run in containers.
* `touch Dockerfile` - make a new Dockerfile
* `FROM` - which image the current image will be based on
* `COPY` - copy files between the filesystem and the image. e.g. `COPY . /var/www/` --> Copy from current directory to /var/www
* `EXPOSE` - expose ports on the image to the outside world. e.g. image with `EXPOSE 80` will listen for traffic on port 80
* `RUN` - image build step. Used to provision the image, the state of the image after the run statement
will be be saved to the image. 
* `CMD` - commands to run on the image after provisioning. It's the command the container runs by default 
when launching the built image. Can have only one `CMD`

#### Running your image
* `docker build -t [image-tag] .` - run in the dir where the dockerfile is located to build the provisioned image.
  * `-t [image-tag]` - give the image a tag (name) for easier running later on
  * `.` - build in current directory. Could be replaced with a path

#### Tagging and pushing to Dockerhub
* `docker tag [image_id] repo/name:latest` - tag the image with repo/name and version
  * `docker login`
  * `docker push [repo/name]` - push the repo to Dockerhub
  * `docker pull [repo/name]` - pull the repo from Dockerhub. Can just do `docker run [repo/name]` and it gets pulled automatically
