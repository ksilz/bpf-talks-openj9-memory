# Eclipse OpenJ9: Memory Diet for Your JVM Applications

## What is this?

[Eclipse OpenJ9](https://www.eclipse.org/openj9/) is a Java Virtual Machine (JVM) that uses less container memory and therefore gives you lower cost. It's mostly a drop-in replacement for the standard [Oracle HotSpot JVM](https://www.oracle.com/java/technologies/javase-downloads.html).

I demonstrated the memory savings in a lightning talk to the [London Java Community](https://www.meetup.com/Londonjavacommunity/) (LJC) on May 1, 2020. Please **[read the slides](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/Eclipse%20OpenJ9%20Memory%20Diet%20-%20LJC%20Lightning%20Talk%202020.pdf)** first! My detailed results are in [an Excel file](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/Detailed%20Results.xlsx).

## Why should I believe you?

You **shouldn't** believe everything you read on the Internet. Granted, I'm a full-stack developer with [21 years of Java experience](https://ksilz.com) and would like to be trustworthy, but still!

That's why you can run OpenJ9 against HotSpot yourselves. And you can tweak them &mdash; change the Java options or even the whole container image!

## What do I need to test OpenJ9 vs HotSpot myself?

You need Docker & Docker Compose. I used two different applications to test OpenJ9 vs HotSpot:

- A [Spring Boot](https://spring.io/projects/spring-boot) web application with a [PostgreSQL database](https://www.postgresql.org), generated with[JHipster](https://www.jhipster.tech)
- 7 benchmarks from the [Rennaissance Suite](https://renaissance.dev)

The containers are limited to 2 GB RAM and 2 CPUs. So I recommend at least 8 GB of RAM and 4 CPU cores to run these applications on your test machine.

## How do I run OpenJ9 vs HotSpot myself?

Each applications has four different Docker Compose Files in the root directory of this repository:

JVM Type | JVM Version | Web application | Benchmarks
---------|-------------|-----------------|----------------
HotSpot  | 8 | [`docker-compose-web-app-hotspot-8.yml`](https://github.com/ksilz/bpf-talks-openj9-memory/blob/master/docker-compose-web-app-hotspot-8.yml) | `docker-compose-benchmark-hotspot-8.yml`
HotSpot | 11 | `docker-compose-web-app-hotspot-11.yml` | `docker-compose-benchmark-hotspot-11.yml`
OpenJ9 | 8 | `docker-compose-web-app-openj9-8.yml` | `docker-compose-benchmark-openj9-8.yml`
OpenJ9 | 11 | `docker-compose-web-app-openj9-11.yml` | `docker-compose-benchmark-openj9-11.yml`

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

This produces tab-separated output. You  can redirect that to a file and then import into a spreadsheet. I did: The benchmark data is in [this folder](https://github.com/ksilz/bpf-talks-openj9-memory/tree/master/results/benchmark) as `data-benchmark-*.txt`. 

## How do I tweak OpenJ9 and HotSpot?

You can tweak both the Java options and the CPUs and memory of the container. For example, here's the Docker Compose 

## How can I build the benchmarks myself?
### Benchmark Suite

````
docker build -t bpf-talks-openj9-memory-benchmark:openj9-11 -f Dockerfile-openj9-11 .
````

### Web Application

````
docker build -t bpf-talks-openj9-memory-web-app:openj9-11 -f Dockerfile-openj9-11 .
````
