---
title: DNS Name Resolution Process
date: 2017-07-12 23:37:49
tags:
categories: "Web学习笔记"
---

Now, suppose you are an employee within XYZ Industries and one of your clients is in charge of the networking department at Googleplex U. You type into your Web browser the address of this department's Web server, “www.net.compsci.googleplex.edu”. In simplified terms, the procedure would involve the following set of steps (Figure 245 shows the process graphically):

* Your Web browser recognizes the request for a name and invokes your local resolver, passing to it the name “www.net.compsci.googleplex.edu”.

* The resolver checks its cache to see if it already has the address for this name. If it does, it returns it immediately to the Web browser, but in this case we are assuming that it does not. The resolver also checks to see if it has a local host table file. If so, it scans the file to see if this name has a static mapping. If so, it resolves the name using this information immediately. Again, let's assume it does not, since that would be boring.

<!--more-->

* The resolver generates a recursive query and sends it to “ns1.xyzindustries.com” (using that server's IP address, of course, which the resolver knows).

* The local DNS server receives the request and checks its cache. Again, let's assume it doesn't have the information needed. If it did, it would return the information, marked “non-authoritative”, to the resolver. The server also checks to see if it has in its zone resource records that can resolve “www.net.compsci.googleplex.edu”. Of course it does not, in this case, since they are in totally different domains.

* “ns1.xyzindustries.com” generates an iterative request for the name and sends it to a root name server.

* The root name server does not resolve the name. It returns the name and address of the name server for the “.edu” domain.

* “ns1.xyzindustries.com” generates an iterative request and sends it to the name server for “.edu”.

* The name server for “.edu” returns the name and address of the name server for the “googleplex.edu” domain.

* “ns1.xyzindustries.com” generates an iterative request and sends it to the name server for “googleplex.edu”.

* The name server for “googleplex.edu” consults its resource records. It sees, however, that this name is in the “compsci.googleplex.edu” subdomain, which is in a separate zone. It returns the name server for that zone.

* “ns1.xyzindustries.com” generates an iterative request and sends it to the name server for “compsci.googleplex.edu”.

* The name server for “compsci.googleplex.edu” is authoritative for “www.net.compsci.googleplex.edu”. It returns the IP address for that host to “ns1.xyzindustries.com”.

* “ns1.xyzindustries.com” caches this resolution. (Note that it will probably also cache some of the other name server resolutions that it received in steps #6, #8 and #10; I have not shown these explicitly.)

* The local name server returns the resolution to the resolver on your local machine.

* Your local resolver also caches the information.

* The local resolver gives the address to your browser.

* Your browser commences an HTTP request to the Googleplex machine's IP address.

![](/images/categories/web/011/dnsresolution.png)

This fairly complex example illustrates a typical DNS name resolution using both iterative and recursive resolution. The user types in a DNS name (“www.net.compsci.googleplex.edu”) into a Web browser, which causes a DNS resolution request to be made from her client machine’s resolver to a local DNS name server. That name server agrees to resolve the name recursively on behalf of the resolver, but uses iterative requests to accomplish it. These requests are sent to a DNS root name server, followed in turn by the name servers for “.edu”, “googleplex.edu” and ‘compsci.googleplex.edu”. The IP address is then passed to the local name server and then back to the user’s resolver and finally, her Web browser software.


Seems rather complicated and slow. Of course, computers work faster than you can read (or I can type, for that matter.) Even given that, the benefits of caching are obvious—if the name was in the cache of the resolver or the local DNS server, most of these steps would be avoided.
