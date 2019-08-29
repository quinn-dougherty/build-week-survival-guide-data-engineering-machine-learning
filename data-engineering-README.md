# build week survival guide: data engineering

1. [overview](#overview)
2. [do's and don'ts](#dodont)
3. [the app](#theapp)
4. [test scripting](#testing)

## [overview](#overview)
1. you are the main _conduit_ between the **back-end engineer(s)** and the
   **machine-learning engineer(s)**
2. the **back-end engineer** is the _conduit_ between **you** and the
   **front-end engineer(s)**

taken together, 1. and 2. mean that the back-end engineer(s) don't necessarily
_need to know_ a lot about what the machine-learning engineer(s) are doing, and
that you don't necessarily need to know a lot about what the front-end
engineer(s) are doing. 

## [do's and don'ts](#dodont)
- _dont_: dive into data cleaning and predictive modeling
- _do_: coordinate with the ML engineer(s), and help them out if they need you and
if you have time. 

- _don't_: wait until every thing fall into place before spinning up your app
- _do_: figure out as much as you can about what the needed scaffolding will be,
and work with dummy IO and beef up your test scripts while you wait for people
to get back to you. 

- _probably don't_: attempt to spin up one app where everything happens at once,
sparking a debate between whether to use javascript or python
- _do_: assume that two apps will be made-- one with a javascript stack, and an API
with a python stack

## [the app](#theapp)
you can write and host however you want, but flask and heroku is basically
standard/most common in BW.

if you're stuck, the following pseudocode may kick-start you
```python
from flask import request
import pickle 
​
@app.route("/swords", methods=['POST'])
def swords():
    ''' a route, expects json object with 2 keys'''
    
    # receive input
    lines = request.get_json(force=True)
    
    # get data from json
    blessed_blade = lines['thunderfury'] # json keys that backend abides
​    ashbringer = lines['corrupted'] 

    # validate input (optional)
    assert isinstance(blessed_blade, int)
    assert isinstance(ashbringer, str)
    
    # deserialize the pretrained model. 
    with open('model.pickle', 'rb') as mod: 
        model = pickle.load(mod)
    
    # predict
    output = model.predict([[blessed_blade, ashbringer]])
    
    # use a dictionary to format output for json
    send_back = {'prediction': output}
    send_back_dummy = {'dummy': 1} # minimal functionality for testing
    send_back_input = { # verify that input is working as expected
        'blessed_blade': blessed_blade, 
        'ashbringer': ashbringer
        }
    
    # give output to sender.
    return app.response_class(
        response=json.dumps(send_back),
        status=200,
        mimetype='application/json'
    )
```
You'll want to **comment out anything to do with the model** at first, because
**you'll be writing this app while you wait for the ML engineer(s) to figure out
how to train the model**.

### the json keys
You'll figure out what json keys you'll actually use by 
1. asking the ML engineers what the predictors are 
2. informing the backend engineer what keys you're expecting. 

ideally, you'll write **API usage** like this

```
at the address `my-app.heroku.com`, along the route `swords`, please send the
object `{'thunderfury': 1, 'corrupted': 'a'}

along a 200, you will receive the object {'prediction': 3.14}
```

remember, **when you see a python dictionary, the backend engineer(s) see json**.
Your _dictionary keys_ convey what the _json keys_ ought to be. 

## [test scripting](#testing)

Most of your APIs are not intended to be viewed in the browser. This means that
you shouldn't test (either your local app or your deployment) by going to a
browser and clicking "refresh". Instead, you'll ping it with json to simulate
exactly how the backend engineer(s) will experience it. 

You can learn to use something called Postman to do this with a live, deployed
app. But I recommend the following script pseudocode 
```python

url = "localhost:5000" # if you want to test local
url = "my-heroku-app.heroku.com" # if you want to test deployed 
​
# this assumes that the agreed upon json key is `key` 
val = {'thunderfury': 1, 'corrupted': 'a'} # 'key' is used within a route like a dictionary key
r_success = requests.post(url, data=json.dumps(val))
​
print(f"request responded: {r_success}.\nthe content of the response was {r_success.json()}")
# you'll get a 200 response if the keys align, and something bad if the keys don't align. 
```

You can run the above code in either a notebook or under an `if
__name__=='__main__':` in a .py file. 
