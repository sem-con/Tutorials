# Developing Semantic Containers

This tutorial describes the basic steps to develop a specific Semantic Container by extending the [semantic base container](https://github.com/sem-con/sc-base). Refer to the [Tutorial-Overview](https://github.com/sem-con/Tutorials) for other aspects.

## Clone Semantic Container Template    
For most use cases the following template provides a scaffold to create a specialized Semantic Container: https://github.com/sem-con/template

```console
git clone https://github.com/sem-con/template.git
```

## Adapt the Template

Use the following checklist to make the necessary changes:

1. specify repository and container name in `build.sh`    
2. add any necessary software components in the `Dockerfile`    
    you might also want to add gems in the `Gemfile` - use as starting point the [version from the base container](https://github.com/sem-con/sc-base/blob/master/Gemfile)    
3. implement a custom method for responding to `GET /api/data` by editing `app/helpers/data_access_helper.rb`    
4. if you want to provide additional API endpoints edit `config/routes.rb` and implement the controller in `app/controllers/api/v1`    
5. to provide custom container initialization edit the `script` directory    
6. by convention it is sensible to provide a default `init.trig` and commands for running the container in `test`        

## Build and Test the new Semantic Container

After you have implemented the new functionality the Semantic Container can be built with the following command in the root directory:    

```console
./build.sh
```

This downloads the current base container and applies the provided changes to create derived Semantic Container. Start the container with `docker run -p 3000:3000 repo/sc-name` and start testing.

## Examples    

Here are some links to examples for extending the base container:    
* [sc-seismic](https://github.com/sem-con/sc-seismic): annotates a open data source and makes it avaialable as Semantic Container    
* [sc-sparql](https://github.com/sem-con/sc-sparql): integrate another service and extend the Semantic Container with additional API endpoints    
* [sc-sentinel](https://github.com/sem-con/sc-sentinel): demonstrates asynchronous processing and process pipelines    