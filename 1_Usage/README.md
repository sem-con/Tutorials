# General Semantic Container Usage

## Access an online Semantic Container  

Get the result of all seismic events worldwide in the last 7 days:

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