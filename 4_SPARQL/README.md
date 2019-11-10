# Semantic Container with SPARQL Endpoint

This tutorial introduces the `sc-sparql` Semantic Containers image. Refer to the [Tutorial-Overview](https://github.com/sem-con/Tutorials) for other aspects.

## Start a SPARQL Semantic Container  

Download the *Semantic Container SPARQL base image* from [Dockerhub](https://hub.docker.com/r/semcon/sc-sparql/) with the following command:  

```console
$ docker pull semcon/sc-sparql
```  

To start a SPARQL container the configuration requires [RML](http://rml.io/spec.html) for mapping the available data to RDF. In the following example the seismic data in [`seismic.json`](seismic.json) is mapped with the following RML:

```
    @prefix scs: <http://w3id.org/semcon/ns/seismic#> .
    @prefix rr:     <http://www.w3.org/ns/r2rml#> .
    @prefix rml:    <http://semweb.mmlab.be/ns/rml#> .
    @prefix ql:     <http://semweb.mmlab.be/ns/ql#> .
    @prefix carml:  <http://carml.taxonic.com/carml/> .
    @prefix dcterm: <http://purl.org/dc/terms/> .
    @prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    @prefix xsd:    <http://www.w3.org/2001/XMLSchema#> .
    @prefix wgs:    <http://www.w3.org/2003/01/geo/wgs84_pos#> .
    @prefix geo:    <http://www.opengis.net/ont/geosparql#> .

    @prefix func:   <http://semantics.id/ns/function#> .
    @prefix param:  <http://semantics.id/ns/parameter#> .
    @prefix fnml:   <http://semweb.mmlab.be/ns/fnml#> .
    @prefix fno:    <http://semweb.datasciencelab.be/ns/function#> .

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
            rr:objectMap <#getTime> ;
        ];

        rr:predicateObjectMap [
            rr:predicate scs:magnitude;
            rr:objectMap [
                rml:reference "magnitude" ;
                rr:datatype xsd:float ;
            ];
        ];

        rr:predicateObjectMap [
            rr:predicate scs:magnitudeType;
            rr:objectMap [
                rml:reference "magnitudeType" ;
            ];
        ];

        rr:predicateObjectMap [
            rr:predicate scs:auth;
            rr:objectMap [
                rml:reference "auth" ;
                rr:datatype xsd:string ;
            ];
        ];

        rr:predicateObjectMap [
            rr:predicate scs:sourceId;
            rr:objectMap [
                rml:reference "sourceId" ;
                rr:datatype xsd:string ;
            ];
        ];

        rr:predicateObjectMap [
            rr:predicate wgs:alt ;
            rr:objectMap [ rml:reference "alt" ; ];
            rr:datatype xsd:float ;
        ];

        rr:predicateObjectMap [
            rr:predicate geo:asWKT ;
            rr:objectMap [
                carml:multiTemplate "Point({lat} {long})" ;
                rr:datatype geo:wktLiteral ;
            ];
        ];
    .

    <#getTime>
        rr:termType rr:Literal ;
        rr:datatype xsd:dateTime;
        fnml:functionValue [
            rr:subjectMap [
                rr:template "http://w3id.org/semcon/seismic/{lastUpdate}";
            ];
            rr:predicateObjectMap [
                rr:predicate fno:executes ;
                rr:object func:timeConversion ;
            ];
            rr:predicateObjectMap [
                rr:predicate param:time ;
                rr:objectMap [
                    rml:reference "lastUpdate" ;
                    rr:datatype xsd:string ;
                ];
            ];
            rr:predicateObjectMap [
                rr:predicate param:timeFormat ;
                rr:objectMap [
                    rr:template "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'" ;
                    rr:datatype xsd:string ;
                ];
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

Use the following command to query 10 triples:   

```console
curl -s 'http://localhost:4000/api/sparql/query?r=true&a=http://localhost:3000/api/data&q=SELECT%20%2A%20WHERE%20%7B%20%3Fa%20%3Fb%20%3Fc%7D%20LIMIT%2010' | jq
```

Or use the SPARQL endpoint on port 4040:    

```console
curl http://localhost:4040/rdf/sparql?query=SELECT%20*%20WHERE%20%7B%20?a%20?b%20?c%7D%20LIMIT%2010
```

*Hint: the tool [yasgui.org](http://yasgui.org) can be used as a webfrontend to perform SPARQL queries*

An example for a more complex SPARQL query is the following time based `FILTER` statement:    

```
prefix scs: <http://w3id.org/semcon/ns/seismic#>
prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?point ?lastUpdate
WHERE {
    ?point rdf:type scs:SeismicActivity .
    ?point scs:lastUpdate ?lastUpdate .
    FILTER (?lastUpdate > "2019-10-28T00:00:00"^^xsd:dateTime && ?lastUpdate < "2019-10-28T11:59:59"^^xsd:dateTime)
}
ORDER BY ?lastUpdate
```

Use URL encoding (or *percent-encoding*) to pass this query as parameter:    

```console
curl http://localhost:4040/rdf/sparql?query=prefix%20scs%3A%20%3Chttp%3A%2F%2Fw3id.org%2Fsemcon%2Fns%2Fseismic%23%3E%0Aprefix%20rdf%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0Aprefix%20xsd%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23%3E%0A%0ASELECT%20%3Fpoint%20%3FlastUpdate%0AWHERE%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%0A%09%3Fpoint%20rdf%3Atype%20scs%3ASeismicActivity%20.%0A%09%3Fpoint%20scs%3AlastUpdate%20%3FlastUpdate%20.%0A%20%20%20%09FILTER%20%28%3FlastUpdate%20%3E%20%222019-10-28T00%3A00%3A00%22%5E%5Exsd%3AdateTime%20%26%26%20%3FlastUpdate%20%3C%20%222019-10-28T11%3A59%3A59%22%5E%5Exsd%3AdateTime%29%0A%7D%0AORDER%20BY%20%3FlastUpdate%0A
```

## Improve the SPARQL Tutorial    

Please report bugs and suggestions for new features using the [GitHub Issue-Tracker](https://github.com/sem-con/Tutorials/issues) and follow the [Contributor Guidelines](https://github.com/twbs/ratchet/blob/master/CONTRIBUTING.md).

If you want to contribute, please follow these steps:

1. Fork it!
2. Create a feature branch: `git checkout -b my-new-feature`
3. Commit changes: `git commit -am 'Add some feature'`
4. Push into branch: `git push origin my-new-feature`
5. Send a Pull Request

&nbsp;    

## Lizenz

[MIT License 2019 - OwnYourData.eu](https://github.com/sem-con/Tutorials/blob/master/LICENSE)


