# betterprojectsfaster.com Talk: OpenJ9

## Running The Benchmark

Just running it:
````
docker-compose -f docker-compose-benchmark-openj9-11.yml up
````

Running it and capturing CPU & memory utilization at the same time on Mac:
````
docker-compose -f docker-compose-benchmark-openj9-11.yml up && docker stats --format "{{.Name}}\t{{.MemUsage}}\t{{.CPUPerc}}" > benchmark-openj9-11.txt
````


## Building The Docker Images
You need to user your own repository name. In the 

````
docker build -t joedata/bpf-talks-openj9-memory-benchmark:openj9-11 -f Dockerfile-openj9-11 .
````