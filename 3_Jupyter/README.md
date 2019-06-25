# Accessing Semantic Containers in Jupyter Notebook

This tutorial provides an example on how to access a Semantic Container from within a Jupyter notebook. Refer to the [Tutorial-Overview](https://github.com/sem-con/Tutorials) for other aspects.

## Example Jupyter Notebooks    

Accesing Semantic Containers in a Jupyter Notebook is as easy as accessing any other online resource. The following example reads data from a Semantic Container on `https://vownyourdata.zamg.ac.at:9505` and stores it in the data frame `df`:

```
# Create URL to JSON file 
url = "https://vownyourdata.zamg.ac.at:9505/api/data/plain"

# Load the JSON file into a data frame
df = pd.read_json(url, orient='records')
```

These Jupyter Notebook examples demonstrate different use case on how to work with data and meta-data provided by Semantic Containers:    
* **Visualizing seismic events:** [SEMCON-ZAMG.ipynb](SEMCON-ZAMG.ipynb)    
* **Checking location restrictions in Usage Policy:** [location_policy.ipynb](location_policy.ipynb)    
* **Working wiht Provenance information:** [provenance_visualization.ipynb](provenance_visualization.ipynb)    


*Credit:* Thanks a lot to user [Peb](https://github.com/pebbie) for creating Jupyter Notebooks!