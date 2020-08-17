This note is influenced by this article https://www.ably.io/concepts/websockets

Before websocket,  most common and used technique was Long Polling. Before long polling, client used to do multiple repeated request to server, which is slow for multiple reasons like every time the connection establishment, http handshake, parsing the http header, query for the data etc. 

So, long polling introduced long lived connection! Rather than communicating with the client multiple time, the server keeps client's connection open as long as possible. The co







Here, XMLHttpRequest connection was used to do the communication.

