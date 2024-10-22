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
