---
layout: post
title: "rate limit per ip   nginx"
description: ""
category: 
tags: []
---
{% include JB/setup %}
Request limit per ip in Nginx
=============================
Nginx is a very popular server and over the time people have devised ingenious ways of customizing the performance and security. I am just going to discuss one of the way to enhance security of Nginx so as to prevent Dos attack. We are going to limit number of requests per ip.
In Nginx 0.8.18 and above you have module HttpLimitReqModule, implementation as follows.
In Nginx config (usually at the path '/etc/nginx/nginx.conf')
`
	http{
		..
			limit_req_zone $binary_remote_addr  zone=test1:10m   rate=5r/s;
		..
	server{
		..
			limit_req zone=test1 burst=10 nodelay;
		..
		}
`
**limit_req_zone** has context http and is basically a directive. From above, it gives a directive that a 10 Mb zone,test1, is allocated to handle sessions and average speed for an ip is limited to 5 requests per second.
**limit_req** is directive with context http, server and location, and could be use as per design and requirement. The directive specifies the zone and busts of request and action delay/nodelay. By default burst size is zero and action is delay. Now by burst size means that if number of waiting request allowed. If speed of requests exceeds the rate specified and number of request in waiting exceed the burst size then in case of 'delay' the requests are delayed and in case of 'nodelay' the waiting requests they are completed with the code 503 : "Service Temporarily Unavailable".
Well now if in real world the things were as straight forward, the lessson would have been over, but they are not, so we would discuss how to rate limit the requests per ip with whitelisting.
Here is the implementation:
`
	http{
		..
			geo $mywhitelist {
				default 1;
				#My IPs
				127.0.0.1/32 0;
		..
			}
			map $mywhitelist $req_limit_zone {
				1   $binary_remote_addr;
				0   "";
			}
		..
			limit_req_zone $req_limit_zone=mywhitelist:10m rate=5r/s;
			limit_req zone= mywhitelist burst=10 nodelay;
`
So, from above config, with the use of geo from HttpGeoModule, helps you render based on geo location context, we create a custom map where we assign address to request zone or blank string, so as to disable the rule, so now this is as simple as it looks.
But the catch here is setting values for rate limit and burst size, the rate limit and burst size has to be set as per the requirement and best way to do is analyze the access logs. For rate limit r/m (requests per minute) can also be used based on the requirement .Once you have decided on the limit run an ab test or any other test you would use for bench-marking and analyze the results, and make sure you have everything right.
Finally you can configure this on your server, 
 run a config reload ( sudo nginx -s reload), monitor the traffic for sometime and then you can feel secure ;)
