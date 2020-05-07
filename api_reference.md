# COVIDSIMTEAM COUCHDB API QUICK REFERENCE

CouchDB holds those of our datasets that are best suited to be stored as a document instead of a graph. I have selected it also because we need to be able to audit mutations and changes.

## Starting a Cookie based Auth Session

You need to be authenticated and authorized to make any subsequent requests to this DB. We are not going to use `Basic Authentication` as it is hard on the server to calculate the hash on each API request. We will switch to JWT based auth at a later point when we will have our own Keycloak SSO. 

To get an AuthSession cookie, `POST` a request to the server:

**HTTP:**
```
POST / HTTP/1.1
Host: http://127.0.0.1:5984/_session
Content-Type: application/x-www-form-urlencoded

name=username&password=password
```
**cURL:**
```Bash
curl --location --request POST 'http://127.0.0.1:5984/_session' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'name=username' \
--data-urlencode 'password=password'
```

**Python:**
```Python
import requests

url = "http://127.0.0.1:5984/_session"

payload = 'name=username&password=password'
headers = {
  'Content-Type': 'application/x-www-form-urlencoded'
}

response = requests.request("POST", url, headers=headers, data = payload)

print(response.text.encode('utf8'))
```


You will get the following kind of response with your cookie:

```Markdown
*   Trying 127.0.0.1:5984...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 5984 (#0)
> POST //_session HTTP/1.1
> Host: 127.0.0.1:5984
> User-Agent: curl/7.68.0
> Accept: */*
> Content-Type:application/x-www-form-urlencoded
> Content-Length: 30
> 
* upload completely sent off: 30 out of 30 bytes
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Cache-Control: must-revalidate
< Content-Length: 46
< Content-Type: application/json
< Date: Wed, 06 May 2020 03:24:33 GMT
< Server: CouchDB/3.1.0 (Erlang OTP/20)
< Set-Cookie: AuthSession=YWRtaW46NUVCMjJERjE6Yvj6S1PKV9CpP3qqwb8xDic206o; Version=1; Expires=Wed, 06-May-2020 05:04:33 GMT; Max-Age=6000; Path=/; HttpOnly
< 
{"ok":true,"name":"user","roles":["_researcher"]}
* Connection #0 to host 127.0.0.1 left intact
```


***N.B.*** 
- Please message me for your username and password and the actual address of the remote server.
- Please use a `.env` file or some external config to store the url, username and password and add this file to `.gitignore`. We cannot have this info pushed to our git repository. You may however push an example `.env` file to the repo.
- The session is set to expire in 100 minutes.
- You can get your current session info and cookie again with a `GET` request to the same endpoint.
- I would recommend sending a `DELETE` request to this same endpoint once you are done to invalidate this session.



## `GET`ting data from endpoints

You can now use the `AuthSession` cookie from the previous request

**HTTP:**
```
GET /provinces HTTP/1.1
Host: http://127.0.0.1:5984/covidsimteam
Content-Type: application/x-www-form-urlencoded
X-CouchDB-WWW-Authenticate: Cookie
Set-Cookie: AuthSession=YW5uYTo0QUIzOTdFQjrC4ipN-D-53hw1sJepVzcVxnriEw
```

**cURL:**
```Bash
curl --location --request GET 'http://127.0.0.1:5984/covidsimteam/provinces' \
       --cookie AuthSession=YW5uYTo0QUIzOTdFQjrC4ipN-D-53hw1sJepVzcVxnriEw \
       -H "X-CouchDB-WWW-Authenticate: Cookie" \
       -H "Content-Type:application/x-www-form-urlencoded"
```

**Python:**
```Python
import requests

url = "http://127.0.0.1:5984/covidsimteam/provinces"

cookie = dict(AuthSession='YW5uYTo0QUIzOTdFQjrC4ipN-D-53hw1sJepVzcVxnriEw')
headers = {
  'Content-Type': 'application/x-www-form-urlencoded',
  'X-CouchDB-WWW-Authenticate': 'Cookie'
}

response = requests.request("GET", url, headers=headers, cookies=cookie)

print(response.text.encode('utf8'))
```

## Endpoints

We have the following `GET` endpoints.

```
/covidsimteam/cities
/covidsimteam/city_distances
/covidsimteam/district_distances
/covidsimteam/districts	
/covidsimteam/palikas
/covidsimteam/provinces

/nepal_spatial/borders
/nepal_spatial/districts
/nepal_spatial/dryports
/nepal_spatial/highways
/nepal_spatial/roads_all
/nepal_spatial/roads_major
/nepal_spatial/settlements

/nepal_spatial/gov_borders
/nepal_spatial/gov_districts
/nepal_spatial/gov_provinces
/nepal_spatial/gov_rivers
/nepal_spatial/gov_roads

/nepal_spatial/gov_wards
/nepal_spatial/gov_gapanapas

# n = 1 to 7
/nepal_spatial/gapanapas_province_n
/nepal_spatial/wards_province_n
```

Responses will have `_id`, `_rev` and a JSON array of objects like in the following example:

```JSON
{
    "_id": "provinces",
    "_rev": "1-135983acdc27971db04f9a498e05d338",
    "provinces": [
        {
            "Name of Government Unit": "Province 1",
            "Hospital": 18,
            "Primary Health Care Centers": 40,
            "Health Post": 648,
            "Urban Health Centers": 52,
            "Community Health Unit": 49,
            "Other Health Facilities": 9,
            "Non-public health facilities": 136,
            "Total health facilities (Public and Non-Public)": 952,
            "No of beds": "",
            "Area": 25905,
            "Pop": 4534943,
            "density": 175.0605289,
            "Index": 0.198458944
        },
        {
            "Name of Government Unit": "Province 2",
            "Hospital": 13,
            "Primary Health Care Centers": 32,
            "Health Post": 745,
            "Urban Health Centers": 17,
            "Community Health Unit": 7,
            "Other Health Facilities": 8,
            "Non-public health facilities": 169,
            "Total health facilities (Public and Non-Public)": 991,
            "No of beds": "",
            "Area": 9661,
            "Pop": 5404145,
            "density": 559.3773936,
            "Index": 0.120278046
        },
        {
            "Name of Government Unit": "Province 3",
            "Hospital": 33,
            "Primary Health Care Centers": 43,
            "Health Post": 640,
            "Urban Health Centers": 110,
            "Community Health Unit": 90,
            "Other Health Facilities": 18,
            "Non-public health facilities": 1386,
            "Total health facilities (Public and Non-Public)": 2320,
            "No of beds": "",
            "Area": 20300,
            "Pop": 5529452,
            "density": 272.386798,
            "Index": 0.298402084
        },
        {
            "Name of Government Unit": "Province 4",
            "Hospital": 15,
            "Primary Health Care Centers": 24,
            "Health Post": 491,
            "Urban Health Centers": 52,
            "Community Health Unit": 41,
            "Other Health Facilities": 12,
            "Non-public health facilities": 101,
            "Total health facilities (Public and Non-Public)": 736,
            "No of beds": "",
            "Area": 21856,
            "Pop": 2403757,
            "density": 109.9815611,
            "Index": 0.312011572
        },
        {
            "Name of Government Unit": "Province 5",
            "Hospital": 20,
            "Primary Health Care Centers": 30,
            "Health Post": 570,
            "Urban Health Centers": 68,
            "Community Health Unit": 47,
            "Other Health Facilities": 6,
            "Non-public health facilities": 174,
            "Total health facilities (Public and Non-Public)": 915,
            "No of beds": "",
            "Area": 19707,
            "Pop": 4499272,
            "density": 228.3083168,
            "Index": 0.222258179
        },
        {
            "Name of Government Unit": "Province 6",
            "Hospital": 12,
            "Primary Health Care Centers": 13,
            "Health Post": 336,
            "Urban Health Centers": 18,
            "Community Health Unit": 22,
            "Other Health Facilities": 3,
            "Non-public health facilities": 60,
            "Total health facilities (Public and Non-Public)": 464,
            "No of beds": "",
            "Area": 30213,
            "Pop": 1570418,
            "density": 51.9782213,
            "Index": 0.382063884
        },
        {
            "Name of Government Unit": "Province 7",
            "Hospital": 14,
            "Primary Health Care Centers": 16,
            "Health Post": 378,
            "Urban Health Centers": 57,
            "Community Health Unit": 43,
            "Other Health Facilities": 3,
            "Non-public health facilities": 45,
            "Total health facilities (Public and Non-Public)": 556,
            "No of beds": "",
            "Area": 19539,
            "Pop": 2552517,
            "density": 130.6370336,
            "Index": 0.274239114
        },
        {
            "Name of Government Unit": "Total",
            "Hospital": 125,
            "Primary Health Care Centers": 198,
            "Health Post": 3808,
            "Urban Health Centers": 374,
            "Community Health Unit": 299,
            "Other Health Facilities": 59,
            "Non-public health facilities": 2071,
            "Total health facilities (Public and Non-Public)": 6934,
            "No of beds": "",
            "Area": 147181,
            "Pop": 26494504,
            "density": 180.0130723,
            "Index": 0.235897981
        }
    ]
}
```

Resources to RTFM™:

*Requests: HTTP for Humans™* [requests.readthedocs.io](https://requests.readthedocs.io/en/master/)
