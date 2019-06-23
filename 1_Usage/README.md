# General Semantic Container Usage

This tutorial introduces the general concept of interacting with Semantic Containers on the command-line. Refer to the [Tutorial-Overview](https://github.com/sem-con/Tutorials) for other aspects.

## Access an online Semantic Container  

We start this tutorial by accessing a public Semantic Container run by ZAMG (Austrian Meteorology and Geophysics Institute) to get the result of all seismic events worldwide in the last 7 days:

```console
$ curl -s "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7"
```  

Use `jq` to have the output nicely formatted:

```console
$ curl -s "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7" | jq
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
$ docker pull semcon/sc-base
```  

The simplest way to start a Semantic Container is through the following command:

```console
$ docker run -p 3000:3000 -d semcon/sc-base
```  

*Note:* make sure to remove the container after usage with `docker rm -f {container name}`; get a list of all running containers with `docker ps`

But usually, you want to provide some additional configuration for the container. In the next example we will provide information about the base image and configure the container with `init_json.trig`.

```console
IMAGE=semcon/sc-base:latest; docker run -d --name test -e IMAGE_SHA256="$(docker image ls --no-trunc -q $IMAGE | cut -c8-)" -e IMAGE_NAME=$IMAGE -p 4000:3000 $IMAGE /bin/init.sh "$(< init_json.trig)"
```  

*Note:* make sure to run the command in the directory with `init_json.trig` present, i.e., `Tutorials/1_Usage/`.

After you have started the container you can also query the empty container:

```console
$ curl -s http://localhost:4000/api/data | jq
```

## Writing data into a Semantic Container

This sections assumes you have started the base image (`semcon/sc-base`) locally on port 4000.

Use the following statement to send data into a Semantic Container:

```console
curl -H "Content-Type: application/json" -d '[{"hello":"world"}]' -X POST http://localhost:4000/api/data
```

Check if the write operation was successfull:  

```console
$ curl http://localhost:4000/api/data/plain
```

You can also create Semantic Container pipelines by reading data from Semantic Container and "piping" the result into another container:

```console
curl -s "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7" | curl -H "Content-Type: application/json" -d "$( cat - )" -X POST http://localhost:4000/api/data
```  

The command above reads all seismic events from the last 7 days (query parameter `duration=7`) and writes the result into a local container on port 4000.

## Display Usage Policy Information for a Semantic Container

The following command displays the usage policy for a given Semantic Container:

```console
curl https://vownyourdata.zamg.ac.at:9500/api/meta/usage
```


## Display Provenance Information for a dataset in a Semantic Container

The following command creates a nicely formatted output of the provenance information. Note that it requires `ruby` to be installed!

```console
curl -s "https://vownyourdata.zamg.ac.at:9702/api/data?file=20190424" | \ 
      jq '.provision.provenance' | ruby -e "puts $(</dev/stdin)"
```