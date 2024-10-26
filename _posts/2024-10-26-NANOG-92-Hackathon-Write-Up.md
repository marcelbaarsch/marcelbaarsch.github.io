---
layout: post
title: "NANOG 92 - Hackathon Write Up"
date: 2024-10-26
categories: [CTF]
---

# Introduction

This hackathon ran virtually along side [NANOG 92](https://nanog.org/events/nanog-92/) in Toronto, Canada. It consisted of CTF style challenges using scenarios built with [containerlab](https://containerlab.dev/) running on pre-provisioned cloud instances. The various scenarios involved network troubleshooting and documentation. 

[NANOG 92 Hackathon](https://nanog.org/events/nanog-92-hackathon/)

## Team Collaboration

Forming teams was optional. I enlisted the help of two of my co-workers who were also attending the conference. Since we all shared the same cloud instance for the challenges, we were able to collaborate using the multiuser mode in `screen`. This setup allowed us to create new windows for logging into specific network devices, as well as watch each other's terminal activity. 

# Scenario: Network Modeling with Nautobot

Information about this scenario can be found on github: [nanog92-hackathon-challenges/scenario_nautobot at main · NANOG/nanog92-hackathon-challenges](https://github.com/NANOG/nanog92-hackathon-challenges/tree/main/scenario_nautobot)

Nautobot is an open-source platform that's meant to be the single source of truth for network data. It's a hard fork of the [NetBox](https://netboxlabs.com/docs/netbox/en/stable/) project. The nautobot server used for this challenge appears to be a [single node in a container lab topology. ]([nanog92-hackathon-challenges/scenario_nautobot/nbhack.clab.yml at main · NANOG/nanog92-hackathon-challenges](https://github.com/NANOG/nanog92-hackathon-challenges/blob/main/scenario_nautobot/nbhack.clab.yml))

## Part 1: Data Model and Web Interface
### Challenge 1.1: Start Nautobot
Containerlab topologies are defined with [YAML](https://en.wikipedia.org/wiki/YAML) configuration files. If your `pwd` is the same as this file, the lab can be deployed using `sudo containerlab deploy`. 

A python environment using [Poetry](https://python-poetry.org/) was required to run the scoring engine that provided the flags necessary to complete the challenges in [CTFd](https://ctfd.io/). 
### Challenge 1.2: Nautobot Basics

The Nautobot web gui could be accessed by connecting to the IP address of the cloud instance through a web browser. It was pre-seeded with data, the being the serial of a device named **leaf2**.
### Challenge 1.4: Create Devices

Instructions were provided to add a new rack and new leaf devices. Running the scoring script provided the flag. 

### Challenge 1.3: Create an API Token

A specific key was provided to create a read/write token.

### Challenge 1.5: IP Addressing and Connections

This was a tutorial on adding IPv4 and IPv6 prefixes to IPAM then setting those addresses and connections on device **leaf3**. The flag was provided by the scoring script.

## Part 2: API Queries

Nautobot is meant to drive network automation so this section uses the REST API to manipulate data. I attempted to write my API Queries in Python with [requests](https://pypi.org/project/requests/). I didn't realize that there's an [existing API client for nautobot](https://docs.nautobot.com/projects/pynautobot/en/latest/) until after the hackathon. The API documentation (http://0.0.0.0/api/docs/) on the Nautobot instance contains a a built in tool to make API queries so the scenario could technically be completed without any scripting. Most of the flags could be obtained with the scoring script.

### Challenge 2.1: API Read Operations

The flag was the integer count
```bash
curl -X 'GET' \
	'http://<IP>/api/dcim/devices/?device_type=cEOS&depth=1' \
	- H 'accept: application/json'
```
### Challenge 2.2: API Create Operations

I started with a script to perform the necessary operation then copying/modifying it to perform subsequent operations. 

```python
import requests

# API endpoint
base_url = 'http://<IP>/api'

# API token
token = '<TOKEN>'

endpoint = '/dcim/device-types/'
url = f'{base_url}{endpoint}'

headers = {
    'Authorization': f'Token {token}',
    'Content-Type': 'application/json'
}

data = {
    "model": "vEOS",
    "manufacturer": "Arista",
    "part_number": "veos",
    "u_height": 2
}

# Send a POST request with JSON data
response = requests.post(url, headers=headers, json=data)

# Check the response
if response.status_code == 201:
    try:
        data = response.json()
        print('Data submitted successfully!')
        print(data)
    except ValueError:
        print("Response is not valid JSON.")
        print(f"Raw response {response.text}")
else:
    print(f"Failed to submit data: {response.status_code}")
    print(f"Error message: {response.text}")
```

```python
import requests

base_url = 'http://<ip>/api'

# API token
token = '<TOKEN>'

endpoint = '/dcim/devices/'
url = f'{base_url}{endpoint}'

headers = {
	'Authorization': f'Token {token}',
    'Content-Type': 'application/json'
}

data = {
    "name": "test-switch",
    "role": "switch_spine",
    "device_type": "active",
    "status": "d77a017e-7791-40e7-8ddf-008ebe5dc485",
    "location": "91dd44c1-de28-42db-85ce-d50a965b93f8"
}

# Send a POST request with JSON data
response = requests.post(url, headers=headers, json=data)

# Check the response
if response.status_code == 201:

    try:
        data = response.json()
        print('Data submitted successfully!')
        print(data)
    except ValueError:
        print("Response is not valid JSON.")
        print(f"Raw response {response.text}")
else:
    print(f"Failed to submit data: {response.status_code}")
    print(f"Error message: {response.text}")
```
### Challenge 2.3: API Update Operations

```python
import requests

# API endpoint
base_url = 'http://<IP>/api'

# API token
token = '<TOKEN>'
endpoint = '/dcim/device-types'
device_type_id = '/430b989a-3bea-47a3-b97c-a4a55dd6f82b/'
url = f'{base_url}{endpoint}{device_type_id}'

headers = {
    'Authorization': f'Token {token}',
    'Content-Type': 'application/json'
}

update_data = {
    "u_height": 1
}

  
# Send a PATCH request with JSON data
response = requests.patch(url, headers=headers, json=update_data)

# Check the response
if response.status_code in [200,201]:
    try:
        data = response.json()
        print('Data submitted successfully!')
        print(data)
    except ValueError:
        print("Response is not valid JSON.")
        print(f"Raw response {response.text}")
else:
    print(f"Failed to submit data: {response.status_code}")
    print(f"Error message: {response.text}")
```

### Challenge 2.4: API Delete Operations

ID values can be found through the web GUI under "advanced". 

```python
import requests

base_url = 'http://<IP>/api'
token = '<TOKEN>'
endpoint = '/dcim/device-types'
device_type_id = '/29d3be7e-adc2-4146-9191-ab2d6520778f/'
url = f'{base_url}{endpoint}{device_type_id}'

# Delete device
device_endpoint = '/dcim/devices'
device_id = '/f7c05eed-d61d-4716-a0cc-513c868bb9af/'
device_url = f'{base_url}{device_endpoint}{device_id}'

headers = {
    'Authorization': f'Token {token}',
    'Content-Type': 'application/json'
}

# Send a POST request with JSON data
response = requests.delete(device_url, headers=headers)
response = requests.delete(url, headers=headers)

# Check the response
if response.status_code == 204:

    try:
        data = response.json()
        print('Data submitted successfully!')
        print(data)
    except ValueError:
        print("Response is not valid JSON.")
        print(f"Raw response {response.text}")
else:
    print(f"Failed to submit data: {response.status_code}")
    print(f"Error message: {response.text}")
```

### Challenge 2.5: Putting it all Together!

Here I tried to refactor my code to use [configparser](https://docs.python.org/3/library/configparser.html) and to implement all CRUD operations in a single script. I had some trouble getting this script to work as expected and I ended up completing this using using the api gui tool. In the future I would do this from the start instead of trying to refactor at the last challenge. 

```python
import requests
import configparser

config = configparser.ConfigParser()
config.read('config.ini')

# Configuration
token = config['DEFAULT']['API_TOKEN']
base_url = config['API']['BASE_URL']
devices_endpoint = config['API']['DEVICES_ENDPOINT']
device_types_endpoint = config['API']['DEVICE_TYPES_ENDPOINT']

headers = {
    'Authorization': f"Token {token}",
    'Content-Type': 'application/json'
}

# API Communication
def nautobot_api_request(method, endpoint, data=None):

    url = f"{base_url}{endpoint}/"

    try:
        if method == 'GET':
            response = requests.get(url, headers=headers)
        elif method == 'POST':
            response = requests.post(url, headers=headers, json=data)
        elif method == 'PATCH':
            response = requests.patch(url, headers=headers, json=data)
        elif method == 'DELETE':
            response = requests.delete(url, headers=headers)
        else:
            return {"error": f"Unsupported HTTP method: {method}"}
        if response.status_code in [200,201]:
            return response.json()
        else:
            return {
                "status_code": response.status_code,
                "error_message": response.text
            }
    except requests.RequestException as e:
        return {"error": str(e)}

# Devices CRUD Functions
def create_device(device_data):
    return nautobot_api_request('POST', devices_endpoint, device_data)

def update_device(device_id, device_data):
    return nautobot_api_request('PATCH', f'{devices_endpoint}/{device_id}', device_data)

def delete_device(device_id):
    return nautobot_api_request('DELETE', f'{devices_endpoint}/{device_id}')

# Device Types CRUD Functions
def create_device_type(device_type_data):
    return nautobot_api_request('POST', device_types_endpoint, device_type_data)
    
def update_device_type(device_type_id, device_type_data):
    return nautobot_api_request('PATCH', f'{device_types_endpoint}/{device_type_id}', device_type_data)

def delete_device_type(device_type_id):
    return nautobot_api_request('DELETE', f'{devices_endpoint}/{device_type_id}')

def challenge2_2():

    device_type_data = {
        "model": "vEOSSS",
        "manufacturer": "Arista",
        "part_number": "veos",
        "u_height": 2
    }

    create_device_type(device_type_data)

# Main function
def main():
    delete_device_type("4591805f-d8d3-4fc5-bb0b-f1b4a836ef39")

if __name__ == "__main__":
    main()
```

# Scenario: Network Troubleshooting With ISIS Over IPv6 And SR-TE

The instructions and topologies were found in this Google Doc: [NANOG 92 hackathon challenges (participants)](https://docs.google.com/document/d/e/2PACX-1vSC9sKP9CoClDPhmy2BsqaRQei9DUWAkzrYWzY2yK1Z-q8tm_yAQBQfnLO5C6KsQflLyKErVsBM28BD/pub)

### Challenge NET-02.1 - ISISv6 Configuration

This topology consisted of four Arista cEOS routers: [network-02.yml](https://github.com/NANOG/nanog92-hackathon-challenges/blob/main/network-02/network-02.yml) 

This documentation from Arista was helpful in troubleshooting the configuration: [EOS 4.32.2F - IS-IS - Arista](https://www.arista.com/en/um-eos/eos-is-is)

There was already some base config on the routers. IPv6 addresses were already configured on the physical interfaces. The loopback 0 (`lo0`) interfaces needed to be configured with an IPv6 address.

This ISISv6 configuration was already present on the boxes. `show isis neighbors` had no results.
```
router isis CORE
   net 49.0001.1921.6800.0001.00
   is-type level-2
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
   !
   traffic-engineering
      no shutdown
      is-type level-2
!
```

In order to get neighbors to come up I needed to enable ISIS routing on the interfaces:
```
r1(config)# interface eth4
r1(config-if-Eth4)# isis enable CORE
```

I also had to enable ISIS routing on the `lo0` interfaces. It was necessary to configure them as passive interfaces since they would not be forming neighbors. 
```
r1(config)# int lo0
r1(config-if-Lo0)# isis passive
r1(config-if-Lo0)# isis enable CORE
```

`show isis neighbors` still wasn't showing any neighbors. I performed the following troubleshooting steps:
- Made sure the net ID for each router was unique
- Made sure the interface addresses were correct and could ping the neighboring interfaces

I noticed there was a warning message regarding no IPv4 being configured on the interfaces when I enabled ISIS routing so I removed it from the router configuration
```
router isis CORE
	no address-family ipv4
```

I got stuck on this challenge for most of the day. I destroyed and re-deployed the container lab topology. Ultimately, it seemed to work after changing the `is-type` from `level-2` to `level-1-2`. 

I was able to confirm ISIS was successfully enabled by seeing two neighbors on each box. I could then ping the loop back interface on a router that wasn't adjacent to the one I was logged into.

The flag was provided by the scoring script which appears to have logged into the routers to check my work.

# Conclusion

Overall, this hackathon demonstrated the potential to extend the CTF format beyond cybersecurity and into network troubleshooting and operations. The ISISv6 configuration challenge especially reminded me of doing labs back when I was enrolled in CCNA trainings at my local college. I'm excited to build my own scenario labs using Containerlab and participating in more CTF competitions in the future! 

Containerlab was also used in a workshop during N92. It appears that this workshop can be run in GitHub Codespaces for free! [srlinuxamericas/N92-evpn: EVPN Workshop at Nanog 92](https://github.com/srlinuxamericas/N92-evpn)
### Final scoring

Our team (Terrabit Carrier Pigeons) finished the hackathon in 4th place. Congrats tk on first place! He took home a Raspberry Pi 5.

![Hackathon leaderboard](/assets/images/nanog-92-hackathon-scoreboard.png)