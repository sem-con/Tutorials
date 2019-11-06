# Semantic Container with SPARQL Endpoint

This tutorial introduces the `sc-sparql` Semantic Containers image. Refer to the [Tutorial-Overview](https://github.com/sem-con/Tutorials) for other aspects.

## Start a SPARQL Semantic Container  

Download the *Semantic Container SPARQL base image* from [Dockerhub](https://hub.docker.com/r/semcon/sc-sparql/) with the following command:  

```console
$ docker pull semcon/sc-sparql
```  

To start a SPARQL container the configuration requires [RML](http://rml.io/spec.html) for mapping the available data to RDF. In the following example the seismic data in [`seismic.json`](seismic.json) is mapped with the following RML:

```
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
@prefix scs:   <http://w3id.org/semcon/ns/seismic#> .
@prefix geo:   <http://www.opengis.net/ont/geosparql#> .
@prefix wgs:   <http://www.w3.org/2003/01/geo/wgs84_pos#> .
@prefix rr:    <http://www.w3.org/ns/r2rml#> .
@prefix rml:   <http://semweb.mmlab.be/ns/rml#> .
@prefix ql:    <http://semweb.mmlab.be/ns/ql#> .
@prefix carml: <http://carml.taxonic.com/carml/> .

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

*Hint: use the [online Turtle validator http://ttl.summerofcode.be/](http://ttl.summerofcode.be/) to check the validity of your mapping*    

The RML is to be provided in the `DataMapping` attributes of the container configuration. A complete configuration [`init_seismic.trig`](init_seismic.trig) is available in this repository and a container can be started with the following command:  

```console
IMAGE=semcon/sc-sparql:latest; docker run -d --name seismic_sparql -e IMAGE_SHA256="$(docker image ls --no-trunc -q $IMAGE | cut -c8-)" -e IMAGE_NAME=$IMAGE -p 4000:3000 -p 4040:3030 $IMAGE /bin/init.sh "$(< init_seismic.trig)"
```  

Use the following command to store data in the container:  

```console
curl -H "Content-Type: application/json" -d "$(< seismic.json)" -X POST http://localhost:4000/api/data
```

## Working with RDF data  

Use the following command to query a triple:   

```console
curl -s 'http://localhost:4000/api/sparql/query?r=true&a=http://localhost:3000/api/data&q=SELECT%20%2A%20WHERE%20%7B%20%3Fa%20%3Fb%20%3Fc%7D%20LIMIT%2010' | jq
```

Or use the SPARQL endpoint on port 4040:    

```console
curl http://localhost:4040/rdf/sparql?query=SELECT%20*%20WHERE%20%7B%20?a%20?b%20?c%7D%20LIMIT%2010
```

*Hint: the tool [yasgui.org](http://yasgui.org) can be used as a webfrontend to perform SPARQL queries*
