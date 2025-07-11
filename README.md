# This is a fork!

Forked from [ail-typo-squatting](https://github.com/typosquatter/ail-typo-squatting) library.

## Requirements

- [requests](https://github.com/psf/requests)
- [flask](https://flask.palletsprojects.com/en/2.1.x/)
- [flask-restx](https://github.com/python-restx/flask-restx)
  - [werkzeug](https://github.com/pallets/werkzeug/)
- [ail-typo-squatting](https://github.com/typosquatter/ail-typo-squatting)
- [idna](https://github.com/kjd/idna)
- [redis](https://github.com/redis/redis-py)
- [tldextract](https://github.com/john-kurkowski/tldextract )
- [pymisp](https://github.com/MISP/PyMISP)
- [beautifulsoup4](https://pypi.org/project/beautifulsoup4/)
- [nltk](https://github.com/nltk/nltk)
- [scikit-learn](https://github.com/scikit-learn/scikit-learn)

## How to install and run

1. Install dependencies `pip install -r requirements.txt`

2. Update configs
   Copy `conf/conf.cfg.sample` to ``conf/conf.cfg`.

3. Install Redis locally and run it
   `brew install redis &  brew services start redis`

4. Run it (make sure your Redis is still running)
   `python Flask_server.py`

## Known issues

There are a few algorithm that requires additional dictionaries that aren't supported out of the box yet.

Including `CommonMisspelling`, `Homoglyph` and `Homophones`. We need to find our own dictionaries for it.

For `WrongTld`, you can download TLD file from the official IANA website by doing this.
```
curl -o tlds-alpha-by-domain.txt https://data.iana.org/TLD/tlds-alpha-by-domain.txt
mkdir -p /usr/local/lib/python3.11/site-packages/etc
mv tlds-alpha-by-domain.txt /usr/local/lib/python3.11/site-packages/etc/
```

## Choice of algorithm

In the `Advanced` option, it's possible to choose algorithms to generate variations from a domain name.

There's 20 algorithms which can be chosen. List can be found [here](https://github.com/typosquatter/ail-typo-squatting#list-of-algorithms-used).

## Download results

After a search, the list of variations can be download to be reused, and also the result can be download to Json format. 

## Misp feed-json

After a search, under the `download` button, it'll be possible to download a Misp event at Json format or go to the page of misp-feed.

## Share

It's possible to share the session by copied the url given by the `share` button who will spawn at the end of a search.

**Notice**: a session is keep for 1h at the moment

## Session db

When a search is done, results are store into a redis db in order to be able to share a session or to be faster the second time a domain is search.

- `session_uuid`:
  - `url` : domain
  - `result_list` : all the result of dns query (Will be remove)
  - `variations_list` : list of variations (Useful for status)
  - `stopped` : boolean to know if the session was stopped (Useful in case someone else search for the same domain)
  - `md5url` : md5sum of the domain
  - `request_algo` : list of algo used for the search (Useful to return only result of the session)
- md5 of the domain: used to know if the domain was already searched
- `md5_of_domain:algo` : list of the result for a given algo (Used in case different people search for the same domain with different algo)
- `event_manifest:session_uuid` : manifest of event
- `event_hashes:session_uuid` : hashes of event
- `event_json:session_uuid` : json of event

## API

This will work only if **`Flask_api.py`** is running.

First, you need to get your *sid* passing the address you want to analyse:

```
curl http://localhost:7006/scan/<url>
```

Second, result from dns check can be obtain using:

```
curl http://localhost:7006/domains/<sid>
```

### Parameters

By passing parameters to the request, it's possible to choose algorithms to run.

```
curl "http://localhost:7006/scan/url?charom&add"
```

- `runAll`

- `addDash`

- `addTld`

- `addition`

- `changeDotDash`

- `changeOrder`

- `commonMisspelling`

- `doubleReplacement`

- `homoglyph`

- `homophones`

- `missingDot`

- `omission`

- `repetition`

- `replacement`

- `singularPluralize`

- `stripDash`

- `vowelSwap`

- `wrongTld`

### Output

```json
[
{"circl.lu":{"A":["185.194.93.14"],"AAAA":["2a00:5980:93::14"],"MX":["10 cppy.circl.lu."],"NS":["ns3.eurodns.com.",...],"NotExist":false,"geoip":"Luxembourg"}}, ...
{"complete":1535,"id":"3322fa4f-52a0-43cb-a057-22bc07bdde01","registered":2,"remaining":4372,"total":5907} 
]
```

The status of the current scan can be found at the end of the json output with : 

`complete`: Number of variations check

`id`: id of the current scan

`registered`: Number of variations which can be accessible with dns

`remaining`: Number of variations  to check to finish the scan

`total`: Number of variations generated
