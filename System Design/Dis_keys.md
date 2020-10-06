### Jargon

#### POP

Point of Presence. 210 worldwide on Dec 5, 2019. Increasing rapidly. 34 countries, 77 cities. Terminates  **TLS handshake** closer POP to the end user. Reuse connection from POP to origin. POP usually has 3 layers of Cache: 

- L1- Load balancer & hot object cache.
- L2- Cache with Layer
- L3- Persistent connection pool for reusing. 

It uses consistent hashing for Peer selection and AWS's private global network for faster communication. 

For Dynamic content, it terminates TLS connection at the edge to improve overall latency, **skips cache layer** and directly goes to L3 to use the connection pool. L3 persistent connection pool improves **latency** since it saves 1.5 round trip for handshake phase. Moreover, it gets better throughput for Maximum **TCP Congestion Window**. 

AWS **Private** Global Network helps to communicate faster. Previously it took *550ms* to a round trip from Indonesia to US using multiple ISP. Now, it takes *70ms* to connect to nearest POP @ Singapore. From Singapore to US in private network takes *215ms*. Total *285ms*. So, it cut the time half already. 

POP caching reduced the origin load dramatically. AWS CloudFront used to serve *12.1 PB* data  daily whereas after implementing POP cache, only *3.6 PB* data was fetched from origin. So, *8.5 PB* data was served from cache. 

AWS developed S2N (new TLS version) with only *6K* line of code compared to OpenSSL which was *500K* line of code. 

