# betterprojectsfaster.com Talk: OpenJ9

## Running The Benchmarks
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


## Building The Docker Images
### Benchmark Suite

````
docker build -t bpf-talks-openj9-memory-benchmark:openj9-11 -f Dockerfile-openj9-11 .
````

### Web Application

````
docker build -t bpf-talks-openj9-memory-web-app:openj9-11 -f Dockerfile-openj9-11 .
````
