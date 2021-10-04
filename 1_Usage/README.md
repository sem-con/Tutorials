# General Semantic Container Usage

*latest update: 4 October 2021*

This tutorial introduces the general concept of interacting with Semantic Containers on the command-line. Refer to the [Tutorial-Overview](https://github.com/sem-con/Tutorials) for other aspects.

## Access an online Semantic Container  

We start this tutorial by accessing a public Semantic Container run by ZAMG (Austrian Meteorology and Geophysics Institute) to get the result of all seismic events worldwide in the last 7 days:

```console
curl "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7"
```  

**Note:** some users reported problems with curl and the used SSL certificate; the following link should work fine in your browser: https://vownyourdata.zamg.ac.at:9500/api/data?duration=7    
With `curl -k` you can ignore SSL problems in curl

Use `jq` to have the output nicely formatted:

```console
curl -s -k "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7" | jq
```  

The default API endpoint to retrieve data from a Semantic Container is `GET /api/data` and the response is a JSON with the following structure:  

```
{  
    "provision": {  
        "content": ["actual data"],  
        "usage-policy": "usage-policy for the data in this container",  
        "provenance": "provenance record about the history of the data"  
    },  
    "validation": {  
        "hash": "SHA256 of the provision section above",  
        "dlt-reference": "reference to distrituted ledger address where the hash value was stored"  
    }  
}
```  


## Running your own Semantic Container

Before you can run your own Semantic Container you need to download the *base image* from [Dockerhub](https://hub.docker.com/r/semcon/sc-base/) with the following command:  

```console
docker pull semcon/sc-base:2021_05
```  

**Note:** we recommend to run a stable version (currently `semcon/sc-base:2021_05`); if you are adventorous you can also try current developer builds by using `semcon/sc-base:latest`    

The simplest way to start a Semantic Container is through the following command:

```console
docker run -p 3000:3000 -d semcon/sc-base:2021_05
```  

**Note:** make sure to remove the container after usage with `docker rm -f {container name}`; get a list of all running containers with `docker ps`

Sometimes, you want to provide some additional configuration for the container. In the next example we will provide information about the base image and configure the container with `init_csv.trig`.

```console
IMAGE=semcon/sc-base:latest; docker run -d --name test -e IMAGE_SHA256="$(docker image ls --no-trunc -q $IMAGE | cut -c8-)" -e IMAGE_NAME=$IMAGE -p 4000:3000 $IMAGE /bin/init.sh "$(< init_csv.trig)"
```  

**Note:** make sure to run the command in the directory with `init_csv.trig` present, i.e., `Tutorials/1_Usage/`.

After you have started the container you can show the current status:

```console
curl -s http://localhost:4000/api/active
```

## Writing data into a Semantic Container

This sections assumes you have started the base image (`semcon/sc-base:2021_05`) locally on port 3000 and did not specify additional configuration options.

Use the following statement to send data into a Semantic Container:

```console
curl -H "Content-Type: application/json" -d '[{"hello":"world"}]' -X POST http://localhost:3000/api/data
```

Check if the write operation was successfull:  

```console
curl "http://localhost:3000/api/data&f=plain"
```

You can also create Semantic Container pipelines by reading data from Semantic Container and "piping" the result into another container:

```console
curl -s -k "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7" | curl -H "Content-Type: application/json" -d @- -X POST http://localhost:3000/api/data
```  

The command above reads all seismic events from the last 7 days (query parameter `duration=7`) and writes the result into a local container on port 3000.

## Display Usage Policy Information for a Semantic Container

The following command displays the usage policy for a given Semantic Container:

```console
curl -k https://vownyourdata.zamg.ac.at:9500/api/meta/usage
```


## Display Provenance Information for a dataset in a Semantic Container

The following command creates a nicely formatted output of the provenance information. Note that it requires `ruby` to be installed!

```console
curl -s -k "https://vownyourdata.zamg.ac.at:9500/api/data" | \
      jq '.provision.provenance' | ruby -e "puts $(</dev/stdin)"
```

## Commit a Container    

There are two options to make data publicly available:    

* running containers can be accessed through the API as described above    
* containers can also be stopped and distributed through images stored in a Docker image repositories (e.g., https://www.dockerhub.com); for data access it is necessary to start the image    
    example commands to stop, publish and run a container / image:    
    ```console
    docker commit my_conainer repo/my_image          # write container to image
    docker push repo/my_image                        # push image to repository
    docker save repo/my_image | gzip > my_image.tgz  # write image to file
    docker load < my_image.tgz                       # read image from file
    docker run -p 3000:4000 repo/my_image            # re-start container
    ```
