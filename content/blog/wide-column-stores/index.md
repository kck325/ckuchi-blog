---
title: Wide column stores
date: "2020-11-27T00:00:00.000Z"
---
There are different kind of databases. To name few:

1. SQL Databases 
2. Key Value Stores
3. Document store
4. Graph Databases
5. Wide column store
6. Column store

This is my attempt to explaining wide column store as ELI5 on wide column stores. The way I would think it as a hybrid between relational and key value stores. You have first level of index on a key which can point to set of rows which look alike. This set of rows are termed as [column family](https://en.wikipedia.org/wiki/Column_family). 

To explain with an example, lets pick up everything store amazon. Amazon sells bunch of items, but all of them are not alike. They can be grouped by categories like apparel, electronics and books. Lets say I buy a lot of clothes and books, and there might be an usecase to display all my data about books for kindle at one place or details about apparel for a different service if you want to store all of those details at one place storing them in a relational store might lead to lot of columns that might not be relavent to different categories. One way to handle that is creating two separate indexes for same key on two categories. You can access the data for catalog by accessing my account id as key and then data that belongs to same category as a set of rows. These group of rows is termed as column family. So in the above example you end up having two column families for same account id. The rows in column family can be access as a relational database. In real world we used this to store data related to cellular usage of wireless users at Twilio. A user can have usage on SMS, Calls or Data. We used column family database to store the usage in cassandra.

account_id: chandra_kuchi

[apparel (column family1)](https://www.notion.so/1b0866d0beb84a7e8006bd08b05f7679)

[books (column family2)](https://www.notion.so/90ea8e7ef399463aa922d64c9b602234)

You might ask what about providing aggregates on summary page. That could be another column family if you wish. But my best guess is you would end up storing those kind of details in another store depending on the query patterns.