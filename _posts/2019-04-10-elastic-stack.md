---
layout: post
title: Elastic Stack
---

1. toc
{:toc}

# Top-down Architecture

Elastic Stack is called ELK previously. Due new products like Beats and X-Pack, the acronym is changed accordingly.

![elastic-stack](/assets/elastic-stack.png)

1. Kibana

   The frontend of the products stack: virtualize data like charts and tables.
2. Elasticsearch

   Jason-based search engine: index, analyze and store data.
3. Elastic Stack Features

   Formerly known as X-Pack.

   A pack of extensions to Kibana and Elasticsearch, like authentication, monitoring & altering, reporting, Elasticsearch SQL, and machine learning (abnormality detection, forecasting, relation graph).

   For relation graph, pay attention to the difference between relevance and popularity. Mostly, relevance is what we desire. All people use Google search in a team does not mean they are relevant as per Google: it is just becuase Google search is popular.
4. Logstash

   Data collection pipeline: unify and normalize data.

   The name comprises 'log' and 'stash'. 'log' means the *input* (source data) while 'stash' means *output* (store the data). Between the input and output, there exists the *filter* component. Congfiguration file format is *similar* to Json.

   >logstash [vt] to store in a usually secret place for future use 

   Logstash has now evolved into a more general tool. It can receive data from and/or send data to Kafka.
5. Beats

   Agent: ships different types of data from edge devices to Logstash or directly to Elasticsearch.

   There are different kinds of beats desginated for different kinds of data. For example, Filebeat is to collect log files (i.e. *access.log* and *error.log*) with modules. A Nginx module collects Nginx log files; a MySQL module collects MySQL log files.

   Another beat is Metricbeat that collects system-level and/or service-level performance metrics like CPU and memory usage for the operating system. It also ships with modules for popular services such as Nginx, MySQL etc.
