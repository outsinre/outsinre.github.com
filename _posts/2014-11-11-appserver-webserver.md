---
layout: post
title: App server VS. Web server
---

>Describe the difference between application server and web server. Pay due attention to their correlation as well.

Most of the times these terms Web Server and Application server are used interchangeably.

Following are some of the key differences in features of Web Server and Application Server:

1. Web Server is designed to serve HTTP Content. App Server can also serve HTTP Content but is not limited to just HTTP. It can be provided other protocol support such as RMI/RPC
2. Web Server is mostly designed to serve static content, though most Web Servers have plugins to support scripting languages like Perl, PHP, ASP, JSP etc. through which these servers can generate dynamic HTTP content.
3. Most of the application servers have Web Server as integral part of them, that means App Server can do whatever Web Server is capable of. Additionally App Server have components and features to support Application level services such as Connection Pooling, Object Pooling, Transaction Support, Messaging services etc.
4. As web servers are well suited for static content and app servers for dynamic content, most of the production environments have web server acting as reverse proxy to app server. That means while servicing a page request, static contents (such as images/Static HTML) are served by web server that interprets the request. Using some kind of filtering technique (mostly extension of requested resource) web server identifies dynamic content request and transparently forwards to app server

Example of such configuration is Apache Tomcat HTTP Server and Oracle (formerly BEA) WebLogic Server. Apache Tomcat HTTP Server is Web Server and Oracle WebLogic is Application Server.

In some cases the servers are tightly integrated such as IIS and .NET Runtime. IIS is web server. When equipped with .NET runtime environment, IIS is capable of providing application services.

Both terms are very generic, one containing the other one and vice versa in some cases.

* Web server: serves content to the web using http protocol.

* Application server: hosts and exposes business logic and processes.

I think that the main point is that the web server exposes everything through the http protocol, while the application server is not restricted to it.

That said, in many scenarios you will find that the web server is being used to create the front-end of the application server, that is, it exposes a set of web pages that allow the user to interact with the business rules found into the application server.
