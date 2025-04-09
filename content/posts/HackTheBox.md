+++
title = "Hack The Box NZ - Wellington - 0x09 Wellington Meetup: Blast Radius - Cloud vs On-Premise"
date = "2025-04-09"
author = ""
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

This is a blog on a HTB meetup in wellington. These events start with a talk, and then go to pwning a retired HTB lab machine.

<!--more-->

## My disorganised notes.

God Account:
giving user god access makes use password everywhere and when attacker leaks gets pass for everywhere

Wide open storage:
Confidential files on windows fileshre anyone in company can access cuz ACL, attacker compromise user laptop reverse control mechanism neeeded. Gigabytes of data need to be exfiltrated so often gets noticed.

S3 bucket:
S3 bucket left open and listable by anonymous because lots of settings and hard.
Attacker bulk harvest
Isn't detected because object level logging is an optional upgrade

Blast radius of mistakes
OnPrem mistakes are limited to what the compr resource can access, restricted by design.
Clouds benefit work against it when compromised - convenient single GUI for all resources, single user can have ultimate power.
Public access by default makes large scale compromise simpler.

On-Prem - hard outer shell
Missed

Cloud - Identity is the new perimeter
all action start with a login
resrouces accessible by default
need to think of every resource as it's own sec problem


Considerations
Blast radius of each mistake
start secure and loosen as you go(least-priov)
security starts during design - don't fall into the trap of assuming you can secure it before go live
each resource type brings new threats
Other stuff

Designing access control
never assign AdminAccess
AWS perms are based on action + target, minimize at least 1 of these, if not both (ex: s3* but limited to a single bucket)
treat your permissions / role definitions as code, make sure you track changes
Regularly revisit! access creep is a big thing

Designing audit
Keep logs of everythign easy but costs
estab how long you need for
test the logs
consider the full stack

Designing the secrets
Handling secrets in cloud is more ez than on-prem
secrets manager can be used just about everywhere
code can easily be transported between environments even on prem
again need

tips for aws
Set a budget
Turn on cloudtrail audit mod
Create user with limited perms
create s3 bucket that allows limit
EC2 instance not on internet
Static weba pp hosted in s3 bucket, with cloudfront in front of it, without exposing S3.

Stakes in cloud are different to not cloud so be weary.

## Lab Machine

Today we didn't actually get to hack a retired HTB machine because of something HTB did?

So we played one of the online machines called "Dog". 

The user solution is to get a user and password using the file in the github on the website. We can use a cve to get a remote shell on the website. After this we can find the user and use the same password. 

For the root, we run sudo -l to find a command we can run and then use it to run a php script and get /root/root.txt.

Very easy challenge, if it didn't keep giving me the weirdest errors which weren't even my fault ;-;.

## The Event
The event was quite fun. Met some friends and got to talk to them and ate lots of pizza. 

It's unfortunate that I had an exam at the start because I finished some of the talk, but I completed the exam in under 30 minutes so I got to be there for basically the whole thing.