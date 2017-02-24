#Docker
##Basic commands

* `docker ps` - list running containers. add `-a` to list all containers.
* `docker images` - list docker images. all `-a` to list intermediate images
* `docker run [image]` - start a docker container and run the [image] in the container

[DockerHub](http://hub.docker.com) can be used to find images.


###Building your own docker image
####Provisioning your image
* `touch Dockerfile` - make a new Dockerfile
* `FROM` - which image the current image will be based on
* `RUN` - image build step. Used to provision the image, the state of the image after the run statement
will be be saved to the image. 
* `CMD` - commands to run on the image after provisioning. It's command the container runs by default 
when launching the built image. Can have only one `CMD`

####Running your image
* `docker build -t [image-tag] .` - run in the dir where the dockerfile is located to build the provisioned image.
  * `-t [image-tag]` - give the image a tag (name) for easier running later on
  * `.` - build in current directory. Could be replaced with a path

