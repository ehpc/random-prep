# Do CORS work inside curl?

Short answer: No — CORS does not apply to curl.

Why?

CORS is a browser security feature.

It exists to stop web pages (running JavaScript) from calling other origins 
without permission.

curl is not a browser, so:
* It does not check CORS headers
* It does not block requests
* It does not care about Access-Control-Allow-Origin at all
