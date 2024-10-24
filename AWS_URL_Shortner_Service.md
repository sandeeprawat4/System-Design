FR - 
1) Systems takes input as a long URL and returns a shortened URL.
2) The short URL should redirect to the long URL when accessed by any user.

Value added FRs -
1) Custom URL creation support.
2) Analytics on the URL access patterns such as most popular URLs.
3) Expiry for a URL so that the URL is auto expired and is no longer accessible after a fixed period of time.
4) The application should be developed as plugin based architecture ensuring extensibility of the architecture. The system has capability to expose APIs to the third party clients to integrate their applications with our system to generate short URLs.

NFR - 
1) Security, to ensure the system is not exploited by bad actors.
2) High availability of the system, to ensure a high uptime percentage in a year.
3) Observability, to ensure appropriate metrics/alerts are in place for constant monitoring of system health.
4) Low latency for short URL creation and redirection.
5) Datastores should be durable and ensure correctness of the data for the expiry time configured for an URL. The data should reside in the system until the expiry time or explicitly removed by a user.
6) Fault tolerance in the system via mechanisms such as retry handling.
7) Interoperability in the system architecture, as the system operates at high scale and can be broken down into multiple subsystems.How will different subsystems interact with each other?

System Scale - 
Generate short URL from long URL - 1k requests/second
Short URL to long URL redirection - 20k requests/second
Average duration of URL persistence in system - 1 year

System Requirement -
Storage - Which storage should I use considering cost and requirement both? Answer below questions.
What is the expected cost we’ll bear if we use Amazon DynamoDB or Amazon Aurora?
Do we require data partitioning from the start?

Total URLs in 1 year = (1000 requests/second) * 60 (per minute) * 60 (per hour) * 24 (per day) * 365 (per year) = ~31.53 Billion
How will you make sure that 31 billion unique url can be generated? Consider including characters and numbers both.
What is the maximum number of unique URLs generated for a given length with these 62 unique characters(alphanumeric)?
Length(1) = 62^1 = 62
Length(6) = 62^6 = 56.8 Billion
Length(7) = 62^7 = 3.5 Trillion
Length(8) = 62^8 = 218.3 Trillion

what is the required storage space? Considering a simple database schema of storing short URLs, long URLs, expiry time and metadata, we can calculate our storage needs as:
Short URL(7 characters) = 7 bytes
Average long URL(100 characters) = 100 bytes
Expiration date(long) = 8 bytes
Average Metadata(userIP, userPreferences, etc.) = 1 KB
Total = ~1150 Bytes. 
Total storage for 1 year = 31.53 Billion * 1 KB = 29.37 TB

In addition to the above, the system has other storage requirements as well such as:
Maintain cache to improve query performance.
Analytics data.
User authentication database. This can be required if the system offers additional capabilities to logged in users such as custom URL creation, view analytics, etc.


Instance type and no. of instances?
Memory for lambda execution

Basic Architecture - 

![untitled_903596_02](https://github.com/user-attachments/assets/7c6e2608-4cf4-417b-bb0b-a8e13f3916ca)

URL Shortening Algorithm - 
Hash function MD5 can be used but total length could be greater than 7.
We can trim but collision may happen since 2 URLs initial 7 letters could be same and with trimming, uniqueness of the urls wont be available. So we rule out hashing.

We can have global counter generation and do base62 encoding. 

![untitled_903596_03](https://github.com/user-attachments/assets/543efb3e-858b-4e52-a7b7-d7ea0adbacec)

But what if machine goes down which is generating the global counter and what if there are multiple machines? How will you make sure that all machines are incrementing one at a time? In that case, we can let DB generate counter but the drawback is that we need to put a lock whenever there is a new request leading to high latency.
To simplify this, We can assign additional responsibility to the system of pre-generating the short URLs since they are not dependent on the long URL. and this can be a separate service since it does not have any dependency on URL shortening service. Each time with a request, unquie ID generater can respond independently.

Key Generation Service - 
We can use MySQL auto_increment or Postgres serial/sequence to auto increment but it can become single point of failure due to dependency on leader instance(if multiple db instances).
We can take inspiration from Flickr architecture which uses MySQL auto-increment feature with REPLACE INTO query to get a globally unique new id on every new query.

CREATE TABLE `Tickets64` (
  `id` bigint(20) unsigned NOT NULL auto_increment,
  `stub` char(1) NOT NULL default '',
  PRIMARY KEY  (`id`),
  UNIQUE KEY `stub` (`stub`)
) ENGINE=InnoDB
 
 
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();


To increase system availability and avoid a single point of failure, two database servers are used starting with an even and odd number and offset as two to avoid collision as reflected in code snippet below. 

TicketServer1:
auto-increment-increment = 2
auto-increment-offset = 1
 
TicketServer2:
auto-increment-increment = 2
auto-increment-offset = 2

![untitled_903596_04](https://github.com/user-attachments/assets/4ab2e44a-4f7c-45f2-afa4-1d207ec59ec7)

Data Model for KGS - 
1) We can maintain 2 tables - Used and Unused. But it has challenges to maintain sync between them through application code.
2) We maintain single table with boolean flag
3) Maintain table with only ununsed IDs and move allocated ones to archieve table for future use. This approach is simple and effective.

APIs - 
POST /v1/createShortUrl
{
    longUrl,
    customUrl,
    expiry,
    userMetadata
}

GET v1/getLongUrl
{
    shortUrl,
    userMetadata
}
HTTP/1.1 302 Found
Location: https://www.example.com/my/long/url
 
HTTP/1.1 404 Not Found
{
    "error": "short url doesn't exist"
}

System Considerarions - 



