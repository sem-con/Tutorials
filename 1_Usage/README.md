# General Semantic Container Usage

## Access an online Semantic Container  

Get the result of all seismic events worldwide:

```console
$ curl -s "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7"
```

Use `jq` to have the output nicely formatted:

```console
$ curl -s "https://vownyourdata.zamg.ac.at:9500/api/data?duration=7" | jq
```
