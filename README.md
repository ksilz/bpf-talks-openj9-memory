# Eclipse OpenJ9: Memory Diet for Your JVM Applications

## What is this?

[Eclipse OpenJ9](https://www.eclipse.org/openj9/) is a Java Virtual Machine (JVM) that uses less container memory and therefore gives you lower cost. It's mostly a drop-in replacement for the standard [Oracle HotSpot JVM](https://www.oracle.com/java/technologies/javase-downloads.html).

I demonstrated the memory savings in a lightning talk to the [London Java Community](https://www.meetup.com/Londonjavacommunity/) (LJC) on May 1, 2020. Please **[read the slides](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/Eclipse%20OpenJ9%20Memory%20Diet%20-%20LJC%20Lightning%20Talk%202020.pdf)** first! My detailed results are in [an Excel file](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/Detailed%20Results.xlsx).

## Why should I believe you?

You **shouldn't** believe everything you read on the Internet. Granted, I'm a full-stack developer with [21 years of Java experience](https://ksilz.com). Aand think of myself as trustworthy. But don't we all?!

That's why you can run OpenJ9 against HotSpot yourselves. And you can tweak them &mdash; change the Java options or build a a whole new container image!

And now for some shameless self-promotion. I write a blog about how to get [better Java projects faster with JHipster and Docker](https://betterprojectsfaster.com). It's been dormant [since the end of 2019](https://betterprojectsfaster.com/blog/), but I'll pick it up again by June 2020. Spoiler alert: I'll also write about [Flutter](https://flutter.dev), Google's cross-platform UI toolkit for native mobile, web & desktop apps. I built [a mobile prototype with it](https://www.youtube.com/watch?v=dxqA6RhEwdQ&t=1s) last year. 

## What do I need to test OpenJ9 vs HotSpot myself?

You need Docker & Docker Compose. I used two different applications to test OpenJ9 vs HotSpot:

- A [Spring Boot](https://spring.io/projects/spring-boot) web application with a [PostgreSQL database](https://www.postgresql.org), generated with[JHipster](https://www.jhipster.tech)
- 7 benchmarks from the open source [Rennaissance Suite](https://renaissance.dev)

Both applications use the most recent Java 8 and 11 versions from [AdoptOpenJDK](https://adoptopenjdk.net), as of April 2020.

The containers are limited to 2 GB RAM and 2 CPUs. So I recommend at least 8 GB of RAM and 4 CPU cores to run these applications on your test machine.

## How do I run OpenJ9 vs HotSpot myself?

Each applications has four different Docker Compose Files in the root directory of this repository:

JVM Type | JVM Version | Web application | Benchmarks
---------|-------------|-----------------|----------------
HotSpot  | 8 | [`docker-compose-web-app-hotspot-8.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-web-app-hotspot-8.yml) | [`docker-compose-benchmark-hotspot-8.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-benchmark-hotspot-8.yml)
HotSpot | 11 | [`docker-compose-web-app-hotspot-11.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-web-app-hotspot-11.yml) | [`docker-compose-benchmark-hotspot-11.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-benchmark-hotspot-11.yml)
OpenJ9 | 8 | [`docker-compose-web-app-openj9-8.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-web-app-openj9-8.yml) | [`docker-compose-benchmark-openj9-8.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-benchmark-openj9-8.yml)
OpenJ9 | 11 | [`docker-compose-web-app-openj9-11.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-web-app-openj9-11.yml) | [`docker-compose-benchmark-openj9-11.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-benchmark-openj9-11.yml)

So here's how you run the web application with HotSpot 11:

````
docker-compose -f docker-compose-web-app-hotspot-11.yml up
````

And this is how you run the benchmarks with OpenJ9 8:

````
docker-compose -f docker-compose-benchmark-openj9-8.yml up
````

When the benchmarks are done, the container stops. That's why I used the `time` command on the Mac to measure how much CPU time they took:

````
time docker-compose -f docker-compose-benchmark-openj9-8.yml up

[...]

benchmark-openj9-8 exited with code 0
docker-compose -f docker-compose-benchmark-openj9-8.yml up  0.73s user 0.12s system 0% cpu 7:50.92 total
````

The web application runs until you manually stop the container. So I only took the time that Spring Boot self-reports in the log:

````
Started SimpleShopApp in 18.302 seconds (JVM running for 19.362)
````

I used the second number in my slides: `JVM running for 19.362`

For both applications, I monitored CPU & memory utilization with `docker stats`: 

````
docker stats --format "{{.Name}}\t{{.MemUsage}}\t{{.CPUPerc}}"
````

This produces tab-separated output. You  can redirect that to a file and then import into a spreadsheet. I did: The benchmark data is in [this folder](https://github.com/ksilz/bpf-talks-openj9-memory/tree/master/results/benchmark) as `data-benchmark-*.txt`. The log output of the benchmark runs is in the same folder as `output-benchmark-*.txt`

## How do I tweak OpenJ9 and HotSpot?

You can tweak both the Java options and the CPUs and memory of the container right in the Docker Compose files. For example, here's [the Docker Compose file for running the benchmark in OpenJ9](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-benchmark-openj9-8.yml):

````
version: '2.2'

services:
  benchmark-openj9-8:
    image: 'joedata/bpf-talks-openj9-memory-benchmark:openj9-8'
    container_name: benchmark-openj9-8
    environment:
      - >-
        JAVA_OPTS=-Xmx1024m -Xms256m -Xtune:virtualized
      - BENCHMARKS=-r 5 gauss-mix,movie-lens,akka-uct,fj-kmeans,mnemonics,scala-doku,finagle-chirper
    cpus: 2.0
    mem_limit: 2147483648
````

You can change the Java options in the `JAVA_OPTS` line. Which Java options can you use?
- For OpenJ9, check [this link for switching from HotSpot](https://www.eclipse.org/openj9/docs/cmdline_migration/). And [this page](https://www.eclipse.org/openj9/docs/cmdline_specifying/) provides links to more information on system properties and command line options in OpenJ9.
- For HotSpot, Java 8 command line options [are listed here](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html). The Java 11 command line options [are here](https://docs.oracle.com/en/java/javase/11/tools/java.html).

The number of CPUs is in the `cpus:` line, and the memory in the `mem_limit` one (in Bytes).

For the benchmark only, you can tweek the `BENCHMARKS` line. The number of repitions is 5 here (`-r 5`). The comma separated list of benchmarks follows right afterwards. The complete list of benchmarks is on [the Rennaissance Suite web site](https://renaissance.dev/docs). Please note that some of them actually crashed in my environemnt.

## How can I change the Java version?

You can't through the Docker Compose files. I build all the Docker images myself, so the Java version is hard-coded in there. If you want a different version of Java, you need to build the Docker images yourself. 

I used the "Debian Slim JRE" images from AdoptOpenJDK as my base images. Here are their Docker Hub repositories:
- [HotSpot 8](https://hub.docker.com/r/adoptopenjdk/openjdk8)
- [HotSpot 11](https://hub.docker.com/r/adoptopenjdk/openjdk11)
- [OpenJ9 8](https://hub.docker.com/r/adoptopenjdk/openjdk8-openj9)
- [OpenJ9 11](https://hub.docker.com/r/adoptopenjdk/openjdk11-openj9)

You can find my Docker Images on Docker Hub, two:
- [Benchmark](https://hub.docker.com/repository/docker/joedata/bpf-talks-openj9-memory-benchmark)
- [Web Application](https://hub.docker.com/repository/docker/joedata/bpf-talks-openj9-memory-web-app)

### How do I Build the Benchmark Docker image?

- Change into the [`src/benchmark`](https://github.com/ksilz/bpf-talks-openj9-memory/tree/master/src/benchmark) folder.
- This folder has 4 Dockerfile: HotSpot and OpenJ9, Java 8 and Java 11. I'm sure you can tell them apart by their names! 😉
- Download version 0.10.0 of the Renaissance Suite JAR file [from the web site](v0.10.0) and save it in this folder. It's huge: 415 MB. 
- There's [an MD5 checksum](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/src/benchmark/renaissance-mit-0.10.0.jar.md5) in the folder so you can test if the JAR file is ok:
````
md5sum --check renaissance-mit-0.10.0.jar.md5
````
- Edit the first line of the Dockerfile you want to change to the new Java Docker image. Here's that line for OpenJ9 8:
````
FROM adoptopenjdk/openjdk8-openj9:x86_64-debianslim-jre8u252-b09_openj9-0.20.0
````
- Now build the Docker image by running a `docker build` in this directory. Here's how you build the OpenJ9 8 image. Please note that I didn't specify a Docker repository here (before the image name):
````
docker build -t my-memory-benchmark:openj9-8-new -f Dockerfile-openj9-8 .
````
- As the last step, update the Docker Compose file to use your new image. That's the `image:` line in there. For the sample image that you just build in the line above, your Docker Compose file would have this `image:` line:
````
    image: 'my-memory-benchmark:openj9-8-new'
````

### How do I Build the web application Docker image?

- Change into the [`src/web-app`](https://github.com/ksilz/bpf-talks-openj9-memory/tree/master/src/web-app) folder.
- This folder also has 4 Dockerfile: HotSpot and OpenJ9, Java 8 and Java 11.
- Download [the web application JAR file](https://bpfr.blob.core.windows.net/talks/openj9-memory-ljc-2020/simple-shop-1.0.0.jar) and save it in this folder. It's 61 MB. 
- There's also [an MD5 checksum](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/src/web-app/simple-shop-1.0.0.jar.md5) in the folder:
````
md5sum --check simple-shop-1.0.0.jar.md5
````
- Now you can edit the Dockerfile, build it and use it in you Docker Compose files the same way as described in the previous section. 