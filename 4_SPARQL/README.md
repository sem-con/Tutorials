# Semantic Container with SPARQL Endpoint

This tutorial introduces the `sc-sparql` Semantic Containers image. Refer to the [Tutorial-Overview](https://github.com/sem-con/Tutorials) for other aspects.

## Start a SPARQL Semantic Container  

Download the *SPARQL base image* from [Dockerhub](https://hub.docker.com/r/semcon/sc-sparql/) with the following command:  

```console
$ docker pull semcon/sc-sparql
```  

To start a SPARQL container the configuration requires [RML](http://rml.io/spec.html) for mapping the available data to RDF. In the example seismic data in `seismic.json` is mapped with the following RML:

```
<#SeismicMapping>
    rml:logicalSource [
        rml:source [
            a carml:Stream;
        ];
        rml:referenceFormulation ql:JSONPath;
        rml:iterator "$.provision.content.[*]" ;
    ];

    rr:subjectMap [
        rr:template "http://w3id.org/semcon/seismic/{sourceId}" ;
    ];

    rr:predicateObjectMap [
        rr:predicate rdf:type;
        rr:objectMap [ rr:template "http://w3id.org/semcon/ns/seismic#SeismicActivity" ];
    ];

    rr:predicateObjectMap [
        rr:predicate scs:lastUpdate;
        rr:objectMap [ rml:reference "lastUpdate" ; ];
        rr:datatype xsd:dateTime ;
    ];

    rr:predicateObjectMap [
        rr:predicate scs:magnitude;
        rr:objectMap [ rml:reference "magnitude" ; ];
        rr:datatype xsd:dateTime ;
    ];

    rr:predicateObjectMap [
        rr:predicate scs:magnitudeType;
        rr:objectMap [ rml:reference "magnitudeType" ; ];
        rr:datatype xsd:dateTime ;
    ];

    rr:predicateObjectMap [
        rr:predicate scs:auth;
        rr:objectMap [ rml:reference "auth" ; ];
        rr:datatype xsd:string ;
    ];

    rr:predicateObjectMap [
        rr:predicate scs:sourceId;
        rr:objectMap [ rml:reference "sourceId" ; ];
        rr:datatype xsd:string ;
    ];

    rr:predicateObjectMap [
        rr:predicate wgs:alt ;
        rr:objectMap [ rml:reference "alt" ; ];
        rr:datatype xsd:float ;
    ];

    rr:predicateObjectMap [
        rr:predicate geo:asWKT ;
        rr:objectMap [
            carml:multiTemplate "Point(({lat}, {long}))" ;
            rr:datatype geo:wktLiteral ;
        ];
    ];
.
```  

The RML is to be provided in the `DataMapping` attributes of the container configuration. A complete `init_seismic.yml` is available in this repository and a container can be started with the following command:  

```console
IMAGE=semcon/sc-sparql:latest; docker run -d --name seismic_sparql -e IMAGE_SHA256="$(docker image ls --no-trunc -q $IMAGE | cut -c8-)" -e IMAGE_NAME=$IMAGE -p 4000:3000 $IMAGE /bin/init.sh "$(< init_seismic.yml)"
```  

Use the following command to store data in the container:  

```console
curl -H "Content-Type: application/json" -d "$(< seismic.json)" -X POST http://localhost:4000/api/data
```

## Working with RDF data  

Use the following command to query a triple:   

```console
curl http://localhost:4000/rdf/sparql?query=SELECT%20*%20WHERE%20{%20?a%20?b%20?c}%20LIMIT%2010 
```

