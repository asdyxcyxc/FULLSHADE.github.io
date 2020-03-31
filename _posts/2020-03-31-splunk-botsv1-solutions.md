---
layout: single
title: Boss Of The SOC - CNIT 50 (BOTSV1) solutions
---

Solutions for the challenges given from Sam Bownes CNIT 50 Splunk class.

https://samsclass.info/50/proj/p2xbots.htm

----

## Level 1: Finding Attack Servers (35 pts)

### BOTSv1 1.1: Scanner Name (5 pts)

Find the brand name of the vulnerability scanner, covered by a green box in the image above.

**Solution**

```
Provided is the task of finding the vulnerability scanners brand name,
you can simply add "Vulnerability" to you search query to find any captured
headers that have that scanner in it
```

**Answer**

`Acunetix`

**Full query**

`index=botsv1 sourcetype="stream:http" imreallynotbatman.com Vulnerability`


### BOTSv1 1.2: Attacker IP (5 pts)

Find the attacker's IP address.

**Solution**

```
Select the src_ip field
```

**Answer**

`40.80.148.42`

**Full Query**

`index=botsv1 sourcetype="stream:http" imreallynotbatman.com Vulnerability | dedup src_ip| table src_ip`


### BOTSv1 1.3: Web Server IP (5 pts)

Find the IP address of the web server serving "imreallynotbatman.com".

```
The source dest_ip field is where the attacker is sending data to
```

**Answer**

`192.168.250.70`

### BOTSv1 1.4: Defacement Filename (10 pts)

Find the name of the file used to deface the web server serving "imreallynotbatman.com".

Hints:

    It was downloaded by the Web server, so the server's IP is a client address, not a destination address.
    Remove the filter to see all 9 such events. Examine the uri values. 

**Solution**

```
Since the attacker is downloading the defacement image onto the server,
the src_ip field is the webserver and not the attackers box.
```

**Answer**

`poisonivy-is-coming-for-you-batman.jpeg`

### BOTSv1 1.5: Domain Name (10 pts)

Find the fully qualified domain name (FQDN) used by the staging server hosting the defacement file.

Hints:

    Examine the 9 events from the previous challenge. Look at the url values. 
    
**Answer**

`prankglassinebracket.jumpingcrab.com`

----

https://samsclass.info/50/proj/p2xbots.htm

----

## Level 2: Identifying Threat Actors (50 pts)

### BOTSv1 2.1: Staging Server IP (10 pts)

In Level 1, you found the staging server domain name (used to host the defacement file). Find that server's IP adddress.

Hints:

    Search for HTTP GET events containing the target FQDN. 
    
**Solution**

```
When filtering by GET requests, and viewing the raw data, you can see the desp_ip
```

**Answer**

`23.22.63.114`

### BOTSv1 2.2: Leetspeak Domain (10 pts)

Use a search engine (outside Splunk) to find other domains on the staging server. Search for that IP address. Find a domain with an name in Leetspeak (like "1337sp33k.com").

**Solution**

```
If you look up the malicious domain, threadcrowd pulls other domains associated with it,
giving the answer
```

**Answer**

`po1s0n1vy.com`

### BOTSv1 2.3: Brute Force Attack (15 pts)

Find the IP address performing a brute force attack against "imreallynotbatman.com".

Hints:

    Find the 15,570 HTTP events using the POST method.
    Exclude the events from the vulnerability scanner.
    Examine the form_data of the remaining 441 events. 

**Solution**

```
Filter by POST requests, and use "NOT Vulnerabilty" to filter out the vulnerability
scanner, finally dedup and create a table of IPs pulled from src_ip
```

**Full query**

`index=botsv1 imreallynotbatman.com AND POST NOT Vulnerability form_data | dedup src_ip | table src_ip`

**Answer**

`23.22.63.114`


### BOTSv1 2.4: Uploaded Executable File Name (15 pts)

Find the name of the executable file the attacker uploaded to the server.

Hints:

    Find the 15,570 HTTP events using the POST method.
    Exclude the events from the vulnerability scanner.
    Search for common Windows executable filename extensions. 

**Solution**

```
Filter your search query with adding *.exe
```

**Answer**

`3791.exe`
