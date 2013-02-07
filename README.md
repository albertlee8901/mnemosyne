Mnemosyne [![Build Status](https://travis-ci.org/johnnykv/mnemosyne.png?branch=master)](https://travis-ci.org/johnnykv/mnemosyne)
=========
## About
Mnemosyne has three main objectives:

1. Provide immutable persistence for [hpfeeds](https://redmine.honeynet.org/projects/hpfeeds/wiki).
2. Normalization of data to enable sensor agnostic analysis.
3. Expose the normalized data through a RESTful API.

## Channels
Mnemosyne currently supports normalization of data from the following channels:

* dionaea.capture
* mwbinary.dionaea.sensorunique
* kippo.sessions
* glastopf.events
* glastopf.files
* thug.events
* thug.files

## Preliminary REST API

Can be found at [http://johnnykv.github.com/mnemosyne/WebAPI.html](http://johnnykv.github.com/mnemosyne/WebAPI.html)

## Speciality services

Mnemosyne is currently serving a speciality service, which on one side collects live dorks from the [Glastopf](https://github.com/glastopf/) [hpfeed](https://redmine.honeynet.org/projects/hpfeeds/wiki), and on the other side correlate the collected data, which allows virgin [Glastopf](https://github.com/glastopf/) instances to bootstrap themselves by using a Mnemosyne service.

# Example queries with curl

### Login to the mnemosyne webservice with the provided credentials
``` bash
curl -k -c cookies.txt -d "username=james&password=bond" https://mnemosyne.honeycloud.net:8282/login
[~]$ cat cookies.txt 

mnemosyne.honeycloud.net  FALSE	/	TRUE	2147487247	beaker.session.id	1f7x19deadbeef8f802fbabe18f1f01a
```


### Malicious websites
Searching for malicious websites in the .ru TLD using a regex. The result of this query is list of urls and hashes of binaries extracted from those urls.
At the moment most data in /urls is generated by [Thug](https://github.com/buffer/thug).
``` bash
curl -k -b cookies.txt "<...>/api/v1/urls?url_regex=\.ru(\/|\:|$)" | python -mjson.tool
```
``` json
{
    "url": "http://xxxyyy.ru:8080/forum/links/django_version.php",
    "_id": "510c35bfc6b6082a30d50bba", 
    "extractions": [
        {
            "hashes": {
                "md5": "f3a80b8b26579a4bfc591f834c767d15", 
                "sha1": "22ff183597009e64ffc3d93f5bccfdb214d6a4bd", 
                "sha512": "63cc541a0743c95037da2fe86cfddc881ad0128665171c50958142f2d8b87a3e90c3085286f774aceb15e28d515ed7b24e399b19579f42da894da206945fe023"
            }, 
            "timestamp": "2013-01-11T15:57:24.107000"
        }, 
        {
            "hashes": {
                "md5": "b33310b69e12427d09e0123c385b3d47", 
                "sha1": "aed4552dfe85ed33f0b77a28abc48c9f831623c5", 
                "sha512": "ab2e1edb49a864c43864f432b0f18a4f097e90b262e4e2964814e022218ad2128a1cdb0402ee76eb75b382f53d145ae7ebd64d0bec7cd599d45ae9c799802b68"
            }, 
            "timestamp": "2013-01-11T15:57:24.107000"
        }, 
        {
            "hashes": {
                "md5": "1f17127f63c975e28710739092117676", 
                "sha1": "64da64d6d6aac36323d326282093e64c89dada40", 
                "sha512": "210055a3a8614da004d92ba6edda8222d483d5563789f9443bb9d5c06481b9674fdf0eef1929410e8c44f9d66c2c7c07e0109e98a5eb92f326a3e8130801f4e7"
            }, 
            "timestamp": "2013-01-11T15:57:24.107000"
        }
    ], 
    "hpfeeds_ids": [
        "50f028a009ce4533628b1af7"
    ]
}
```

### Looking for files
``` bash
curl -k -b cookies.txt "<...>/api/v1/files?hash=b420138b88eda83a51fea5298f72864a" | python -mjson.tool
```
```json
{
    "files": [
        {
            "_id": "510c3ce5c6b6082a30d548f3", 
            "content_guess": "PE32 executable (DLL) (GUI) Intel 80386, for MS Windows, UPX compressed", 
            "data": "4d5a90000300000004000000ffff0000b8 <--- SNIP! --->", 
            "encoding": "hex", 
            "hashes": {
                "md5": "b420138b88eda83a51fea5298f72864a", 
                "sha1": "0e644fc39a287e6f020ede6d6c9dd708b1a871ba", 
                "sha512": "98a2110f389790b5fd66f50e26e85465b3d22662245969b1fd03025194ef7a00a928c3709b57e20d165876231cdab12d38b7ff17e5c173b6562e924dc4087d85"
            }, 
            "hpfeed_ids": [
                "50f3e41b09ce4533629cea00"
            ]
        }
    ]
}
```
### Overview of the file store
``` bash
curl -k -b cookies.txt "<...>/api/v1/files/types" | python -mjson.tool
```
```json
{
    "content_guesses": [
        {
            "content_guess": "Javascript", 
            "count": 1182
        }, 
        {
            "content_guess": "PE32 executable (DLL) (GUI) Intel 80386, for MS Windows", 
            "count": 481
        }, 
        {
            "content_guess": "PE32 executable (DLL) (GUI) Intel 80386, for MS Windows, UPX compressed", 
            "count": 732
        }, 
        {
            "content_guess": "MS-DOS executable, MZ for MS-DOS", 
            "count": 1
        }, 
        {
            "content_guess": "PE32 executable (GUI) Intel 80386, for MS Windows", 
            "count": 46
        }, 
        {
            "content_guess": "Applesoft BASIC program data", 
            "count": 5
        },
        {
            "content_guess": "GIF image data, version 89a, 16129 x 16129", 
            "count": 31
        }, 
        {
            "content_guess": "DOS executable (COM)", 
            "count": 2
        },
        {
            "content_guess": "PE32 executable (GUI) Intel 80386, for MS Windows, UPX compressed", 
            "count": 3
        },
        
        <--- SNIP! --->
    ]
}
```
### Dorks
Dorks collected by [Glastopf](https://github.com/glastopf/glastopf)
``` bash
curl -k -b cookies.txt "<...>/api/v1/aux/dorks?limit=10" | python -mjson.tool
```
```json
{
    "dorks": [
        {
            "content": "/pivotx/includes/index.php", 
            "count": 716, 
            "firsttime": "2013-02-01T20:38:42+00:00", 
            "lasttime": "2013-01-14T16:20:51.504000", 
            "type": "inurl"
        }, 
        {
            "content": "/axis-cgi/mjpg/wp-content/themes/diner/timthumb.php", 
            "count": 545, 
            "firsttime": "2013-02-01T20:38:32+00:00", 
            "lasttime": "2013-01-14T16:26:03.036000", 
            "type": "inurl"
        }, 
        {
            "content": "/board/board/include/pivotx/includes/wp-content/pivotx/includes/timthumb.php", 
            "count": 493, 
            "firsttime": "2013-02-01T20:39:03+00:00", 
            "lasttime": "2013-01-14T10:55:50.197000", 
            "type": "inurl"
        },
        
        <--- SNIP --- >
        
         ]   
}
```

### Sessions
Searching for all honeypot attacks comming from an specific source port.
``` bash
curl -k -b cookies.txt "<...>/api/v1/sessions?source_port=37337" | python -mjson.tool
```
```json

{
  "sessions": [
    {
          "_id": "510c2f1209ce45385d3ed584", 
          "honeypot": "dionaea", 
          "attachments": [
              {
                  "description": "Binary extraction", 
                  "hashes": {
                      "md5": "984cef500b81e7ad2f7a69d9208e64e6", 
                      "sha512": "e899155228a1d3b5ed9864a7fed944716b7b0a3061b76e0f720bf9f7f6c65c633d8fdd4799335b9d92238b4b18e8076718a87a5d7a6538fec4223f111224b5e5"
                  }
              }
          ], 
          "destination_ip": [
              "xxx.yyy.zzz.ppp"
          ], 
          "destination_port": 445, 
          "hpfeed_id": "50ec09b709ce451dac5c844e", 
          "protocol": "microsoft-ds", 
          "source_ip": "xxx.yy.xx.xxx", 
          "source_port": 37337, 
          "timestamp": "2013-01-08T11:57:43.390000"
      }, 
      {
          "_id": "510c2fbc09ce45385d3fcd16",
          "honeypot": "glastopf", 
          "destination_port": 80, 
          "hpfeed_id": "50ec74f509ce452427303b50", 
          "protocol": "http", 
          "session_http": {
              "request": {
                  "body": "", 
                  "header": "{<--- SNIP --->}", 
                  "host": "<--- SNIP --->", 
                  "url": "<--- SNIP --->", 
                  "verb": "GET"
              }
          }, 
          "source_ip": "xxx.zzz.yy.zzz", 
          "source_port": 37337, 
          "timestamp": "2013-01-08T13:28:15"
      },
      ]
}
```



