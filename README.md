# Databases and Indexing

## Abstract
**Databases gets slower with more requests.**

**If the database responds in over 1 second, it might make the user dissatisfied.**

**Indexing makes the database significantly faster.**

**Requests don&#39;t end up being queued so the response time doesn&#39;t get longer and longer**


When we&#39;ve worked on different systems, we&#39;ve noticed the responses of databases slowing down. We discovered that this was due to the excessive data. The more data we needed to request and read, the slower the queries. It makes natural sense, but it doesn&#39;t change that it would ruin the user experience.

Normally, one wouldn&#39;t think a slow query would matter much if it only happened once. However, in a situation where a system needs to read the database several times, it could add up. The read time might not scale linearly either. It could scale exponentially instead. Therefore, there might be a need to find a fix to an eventual problem, and fast.

To put it as simply as possible, read queries taking longer harms the overall experience. The time lost compared to the original query time adds up over time. A query taking 5 seconds normally, then costing 10 would see a 100% increase in useless time. This puts us at the core issue:

**A database gets slower the further it grows, is it possible to reduce the query time while maintaining the size?**

To get our solution, first we need to talk about how a database is typically accessed. For the rest of this entry, we&#39;ll use SQL notations to explain relevant elements.

To access any data in a database, you need to construct a query. Since we&#39;re working with _reading_ the database, our query would look like this:

**SELECT \* FROM TABLE\_NAME**

Running this query starts the table at the top left part of the table. From that point, it continues to the right, reading every entry in the row<sup>1</sup>. This procedure is repeated for however many rows are necessary to fulfill the query. Once this looped procedure reaches its end, the query-fulfilling data is returned to the user.

Databases are not static however, as their connected systems connect to them regularly. This means that there is already one factor that slows down queries: the increasing size. The second factor is the way queries are computed as previously explained. The size of a table&#39;s data is not static, but the queries accessing them are.

Queries are naturally slow, because they always start from the first part of the table. Even if you only want a single specific entry, the query will always start at the top left.

So, this means that the query needs other entry points to finish faster. Keep in mind that this only concerns read-type queries. Insert and delete frequently do not rely on previous data within the table.

So, how do we make new entry points for the queries to utilize? We make use of something called indexing. Indexing can be compared, in layman&#39;s terms, to a table of contents, or a glossary<sup>2</sup>. Essentially, it makes a redundant version of an existing table, ordered by a given criteria. You could have a table full of names, and apply a last\_name, first\_name index. This would then make a table ordered by the last names, in alphabetic order.

We&#39;ve taken one of our own systems, and tried to compare results. First, we ran a query on a large table without any indexes. Then we tried a similar query afterwards, once an index had been made. In the box below, you can see both of the specific queries, and the results.

**Query 1: SELECT TOP 100 \* FROM Comments ORDER BY PublishDate DESC** 

**Query 2: SELECT TOP 100 \* FROM Comments ORDER BY OwnerID DESC**

**Results:** https://imgur.com/a/jMoTYVU

As seen above, there is a massive difference once a table has been indexed. Before, it took around 36183 milliseconds to process the query. After, it took merely 283 milliseconds. To put it bluntly, we managed to save about 36 seconds on one read query. It saves a remarkable amount of time for the average user, if put to use.

The way queries are executed also changes depending on the use of indexing. In the box below, I&#39;ve got a diagram of the two queries. This diagram shows how the system goes about accessing the data. Pay close attention to the percentages below the icons.

**Diagram:** https://imgur.com/a/84BMN5u

As you can see, the elements that the two queries access are astonishingly different. The first query, which is indexed, has an easier time. It merely takes the indexed table and takes the bottom 100 rows. This is because the table is sorted, so it can take 1-100 from the bottom. Because the second query has no indexing applied, it first needs to sort the table. Then, it does the same, taking the bottom 100 rows from the descending table.

This demonstrates how indexing a database table ultimately makes it faster, answering our main question. However, surely it&#39;s not all good? There has to be some sort of downside, otherwise indexing would be used everywhere.

The core problem with indexing is storage issues. The more indexes you apply to a table, the more space you need. If indexes are used in excess, you risk hitting size limits on the file system. This is why indexing is typically only applied to one or two fields per table. You don&#39;t _need_ to be able to order the tables by all criteria. So, take good care of which fields are indexed.

To explain the issue in a simple manner, please read the excerpt below.

 _Simple Example:_
 
 _Consider a &quot;book&quot; of 1000 pages that needs to be divided into sections of 100. This means we&#39;ll have 10 &#39;pages&#39; of data (1000/100). Since we now only have 10 index pages, searching for the desired data is easier. No need to scrub all 1000 pages, just the section that matches the criteria. Once the section is narrowed down, you have a narrower amount of data to examine. This naturally means the desired &#39;page&#39; is found much faster._
 
_Then, in addition to the original 1000 pages, you need to store the index. This means you&#39;re storing 1010 pages, and these add up. So too many indexes can cause issues with the file systems size limits._<sup>3</sup>

Another issue with indexes comes from the way they act as references. Since they refer to a certain amount of data, they update _with_ the table. This means the other three main queries, Add, update and delete take a bit longer. So the time gained on read queries is offset by the time lost. This means that there&#39;s another balancing act to keep track of.

With the positives and negatives of indexing in mind, we support the use of indexing. Typically, we work with systems that don&#39;t experience a lack of storage to support indexing. This means that we can freely add indexes to our databases, without worrying about storage.

However, while this is the case for _our_ systems, it doesn&#39;t mean it&#39;s the best solution for _all_ systems. Keeping the additional storage space necessary to compute the indexes in mind, a question arises. Is the speed gained from the indexes worth running into issues on the storage front? Sometimes, the speed gained isn&#39;t enough to offset the cost. This is why it is always necessary to weigh cost vs gain.
Despite this, we still stand behind our choice. We encourage the use of indexing in databases, so queries can retrieve data faster. However, remember that it is not the only, nor perfect, option. It is merely _an_ option.

## Glossary:

1. [https://stackoverflow.com/a/172992](https://stackoverflow.com/a/172992) - Used as reference to explain how database querying works
2. [https://stackoverflow.com/a/15243748](https://stackoverflow.com/a/15243748) used as reference to explain the basics of indexing
3. [https://stackoverflow.com/a/43572540](https://stackoverflow.com/a/43572540) used as reference to simplify an issue with index.
