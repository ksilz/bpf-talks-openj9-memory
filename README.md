# Eclipse OpenJ9: Memory Diet for Your JVM Applications

## What is this?


## How can I run the Benchmarks Myself?
### Benchmark Suite

Just running it:

````
docker-compose -f docker-compose-benchmark-openj9-11.yml up
````

Running it and capturing CPU & memory utilization at the same time on Mac:

````
time docker-compose -f docker-compose-benchmark-openj9-11.yml up && 
````

Run this at the same time in a different terminal:

````
docker stats --format "{{.Name}}\t{{.MemUsage}}\t{{.CPUPerc}}" > benchmark-openj9-11.txt
````

### Web Application

Just running it:

````
docker-compose -f docker-compose-web-app-openj9-11.yml down && docker container prune -f && docker-compose -f docker-compose-web-app-openj9-11.yml up
````

Capturing memory utilization at the same time on Mac: Run this at the same time in a different terminal:

````
docker stats --format "{{.Name}}\t{{.MemUsage}}"
````


## How can I build the benchmarks myself?
### Benchmark Suite

````
docker build -t bpf-talks-openj9-memory-benchmark:openj9-11 -f Dockerfile-openj9-11 .
````

### Web Application

````
docker build -t bpf-talks-openj9-memory-web-app:openj9-11 -f Dockerfile-openj9-11 .
````
