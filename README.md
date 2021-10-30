# docker-slim Testing

This project aims to showcase and test out the capabilities of [docker-slim](https://github.com/docker-slim/docker-slim/) for several use cases:
- Optimizing a larger, debian based Docker image running a java app.
- Optimizing a smaller, alpine-based Docker image running a python app.

I'd highly recommend checking out `docker-slim` using the link above; I will directly paste the repository's description:
```
Don't change anything in your Docker container image and minify it by up to 30x (and for compiled languages even more) making it secure too! (free and open source)
```

If you are anything like I am, you are probably thinking: "surely this is too good to be true" or "does this only apply to really layered images with many `RUN` statements?". 

Well, let's have a look.

## Dockerfile 1: Containerized Spring (Java) App
First, let's take a quick look at the [Dockerfile](./java/Dockerfile) supporting my hello-world Java app:
```Dockerfile
FROM openjdk:8
EXPOSE 8080
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD ["./mvnw", "spring-boot:run"]
```
When built, it yields an image of the following size:
```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
javatest     latest    dff8c8aa3566   7 seconds ago   520MB
```

Well, that's pretty hefty. Admittedly this could be shaved down by using a smaller alpine-based image, but we'll explore that in the Python example. For now, let's see what `docker-slim` can do for us.

```bash
docker-slim build --target javatest:latest
```
```bash
$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
javatest.slim   latest    76c7a799c027   6 seconds ago   214MB
javatest        latest    dff8c8aa3566   3 minutes ago   520MB
```

So, from running one command and without refactoring the Dockerfile manually, we went from 520MB (with what was a pretty simple Dockerfile) to 214MB (41% of its original size!).

The million dollar question of course is: does the app the image is serving still function?
```bash
evanseabrook@Evans-MacBook-Pro Java % docker run --name javatest -P -d javatest.slim:latest
3c2148270edac2a19a7f1df1c26ab7daa29bae7243a25fe1bfc20a6261f4f9f8
evanseabrook@Evans-MacBook-Pro Java % docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                     NAMES
3c2148270eda   javatest.slim:latest   "./mvnw spring-boot:…"   7 seconds ago   Up 6 seconds   0.0.0.0:55030->8080/tcp   javatest
evanseabrook@Evans-MacBook-Pro Java % curl http://localhost:55030
<p>Hello there</p>%  
```
Yes, yes it does.


## Dockerfile 2: Containerized Flask (Python) App
Well, the results from the JDK image were pretty impressive; let's try something a little more challenging and move onto our [next Dockerfile](./python/Dockerfile):
```dockerfile
FROM python:3.8-alpine3.13
ADD . /api
WORKDIR /api
RUN pip install -r requirements.txt
EXPOSE 80
CMD ["python", "src/main.py"]
```
This time, we are using a much smaller (alpine-based) image, running a bare-bones Flask app.

Let's build the image as-is and see what we have to work with:
```
REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
pythontest   latest    55c25c2b963a   About an hour ago   64.1MB
```
The initial image is 64.1MB -- not a lot to work with there. However, let's still see what `docker-slim` can do for us:
```bash
docker-slim build --target pythontest:latest
```
```bash
evanseabrook@Evans-MacBook-Pro python % docker images
REPOSITORY        TAG       IMAGE ID       CREATED             SIZE
pythontest.slim   latest    4ae57b3849c0   14 seconds ago      18.4MB
pythontest        latest    55c25c2b963a   About an hour ago   64.1MB
```
Starting with an already pretty slimmed down image (64.1MB), we still see a reduction down to 18.4MB (28.7% of its original size).

## Conclusion
Having run through the above two examples, we can say the following of `docker-slim`:
 1. It most certainly does work, and
 2. It greatly optimizes small images that didn't even really need it in the first place.

 Again, you can (and definitely should) check out `docker-slim`'s [GitHub repository](https://github.com/docker-slim/docker-slim/) for a deeper dive into usage, as well as instructions on how to install it. Happy optimizing!