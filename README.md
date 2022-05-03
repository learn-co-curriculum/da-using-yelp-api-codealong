# Using the Yelp API - Codealong

## Introduction

Now that we've discussed HTTP requests and OAuth, it's time to practice applying those skills to a production level API. In this codealong, we'll take you through the process of signing up for an OAuth token and then using that to make requests to the Yelp API!

## Objectives

You will be able to:

* Make requests using OAuth
* Use the JSON module to load and parse JSON documents
* Convert JSON to a pandas dataframe

## Generating Access Tokens

As discussed, in order to use many APIs, one needs to use OAuth which requires an access token. As such, our first step will be to generate this login information so that we can start making some requests.  

With that, let's go grab an access token from an API site and make some API calls!
Point your browser over to this [yelp page](https://www.yelp.com/developers/v3/manage_app) and start creating an app in order to obtain and API access token:

![](./images/yelp_app.png)


You can either sign in to an existing Yelp account or create a new one if needed.

On the page you see above, simply fill out some sample information such as "Flatiron Edu API Example" for the app name, or whatever floats your boat. Afterward, you should be presented with an API key that you can use to make requests!

With that, let's set up our authentication tokens so that we can start making some API calls!

### Should I publicly share my passwords on Github?

When using an API that requires an API key and password you should **NEVER** hardcode theses values into your main file. When you upload your project onto github it is completely public and vulnerable to attack. Assume that if you put sensitive information publicly on the internet it will be found and abused. 

To this end, how can we easily access our API key without opening ourselves up to vulnerabilities?

There are many ways to store sensitive information but we will go with this method. 

#### Create a `.secret/` directory

This will create a new folder in your project directory where you can store files for any of the API information you have. If you were to push this project to GitHub (ie. share it online) we could put this file in our .gitignore so it would not be shared. To this end, you will be the only one with access to this folder and your API key will be secure!


```python
!mkdir .secret/
```

#### Check to make sure it was created

dot files won't show up with just `ls` you must use the show all command as well `ls -a`


```python
!ls -a
```

#### Secure API Key in .json file

Next we will create a `.json` file to store our API key. In the cell below, replace `<YOUR_API_KEY>` with the API key you were given by yelp. **Make sure you do not remove any quotations**.


```python
!echo '{"api_key": "<YOUR_API_KEY>"}' >> yelp_api.json
```

#### Move `yelp_api.json` to `.secret/`

Finally we can move the `.json` file storing our API key into our secure `.secret/` folder. 


```python
!mv yelp_api.json .secret/
```

The function below opens the `.json` file of the given `path`. We have provided the path to your `yelp_api.json` file below.


```python
import json

def get_keys(path):
    with open(path) as f:
        return json.load(f)
```


```python
keys = get_keys("./.secret/yelp_api.json")

api_key = keys['api_key']
```

You may check the `api_key` variable to ensure your API key was imported properly. Once you are sure your API key is correct, it is highly recommended you clear the cell's output and delete the `!echo '{"api_key": "<YOUR_API_KEY>"}' >> yelp_api.json` cell above so your API key is not listed anywhere in this notebook! Again, you don't way your keys stolen!

## An Example Request with OAuth <a id="oauth_request"></a>
https://www.yelp.com/developers/documentation/v3/get_started

In the next lesson, we'll further dissect how to read and translate online documentation like the link here. For now, let's simply look at an example request and dissect it into its constituent parts:


```python
import requests
term = 'Mexican'
location = 'Astoria NY'
SEARCH_LIMIT = 10

url = 'https://api.yelp.com/v3/businesses/search'

headers = {
        'Authorization': 'Bearer {}'.format(api_key),
    }

url_params = {
                'term': term.replace(' ', '+'),
                'location': location.replace(' ', '+'),
                'limit': SEARCH_LIMIT
            }
response = requests.get(url, headers=headers, params=url_params)
print(response)
print(type(response.text))
print(response.text[:1000])
```

## Breaking Down the Request

As you can see, there are three main parts to our request.  
  
They are:
* The URL
* The header
* The parameters
  
The URL is fairly straightforward and is simply the base URL as described in the documentation (again more details in the upcoming lesson).

The header is a dictionary of key-value pairs. In this case, we are using a fairly standard header used by many APIs. It has a strict form where 'Authorization' is the key and 'Bearer YourApiKey' is the value.

The parameters are the filters that we wish to pass into the query. These will be embedded into the URL when the request is made to the API. Similar to the header, they form key-value pairs. Valid key parameters by which to structure your queries are described in the API documentation which we'll look at further shortly. A final important note, however, is the need to replace spaces with "+". This is standard to many requests as URLs cannot contain spaces. (Note that the header itself isn't directly embedded into the URL itself and as such, the space between 'Bearer' and YourApiKey is valid.)


## The Response

As before, our response object has both a status code, as well as the data itself. With that, let's start with a little data exploration!


```python
response.json().keys()
```

Now let's go a bit further and start to preview what's stored in each of the values for these keys.


```python
for key in response.json().keys():
    print(key)
    value = response.json()[key] #Use standard dictionary formatting
    print(type(value)) #What type is it?
    print('\n\n') #Separate out data
```

Let's continue to preview these further to get a little better acquainted.


```python
response.json()['businesses'][:2]
```


```python
response.json()['total']
```


```python
response.json()['region']
```

As you can see, we're primarily interested in the 'businesses' entry. 

Let's go ahead and create a dataframe from that.


```python
import pandas as pd

df = pd.DataFrame.from_dict(response.json()['businesses'])
print(len(df)) #Print how many rows
print(df.columns) #Print column names
df.head() #Previews the first five rows. 
#You could also write df.head(10) to preview 10 rows or df.tail() to see the bottom
```

## Summary <a id="sum"></a>

Congratulations! We've covered a lot here! We took some of your previous knowledge with HTTP requests and OAuth in order to leverage an enterprise API! Then we made some requests to retrieve information that came back as a JSON format. We then transformed this data into a dataframe using the Pandas package. In the next lab, we'll break down how to read API documentation!
