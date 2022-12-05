---
title: "HTTP"
date: 2022-02-23T13:38:42Z
author: ["oaiqiy"]
draft: false
tags: ["http","network"]
categories: ["note"]
---

content from *Diagram HTTP*

## Web and network basics

URI *protocol*://*user*:*pass*@*server address*:*port*/*path*?*search string*#*fragment identifier*

## Simple HTTP

Request structure  
*method* *URI* *version*  
*headers*  
*content*

Response structure
*version* *status code* *reason-phrase*  
*headers*  
*body*

HTTP is a kind of stateless protocol. HTTP/1.1 use Cookie to control status.

### HTTP method

* GET get resource
* POST transfer entity body
* PUT transfer file (unsafe)
* HEAD get message header
* DELETE delete file
* OPTIONS query URI supporting methods
* TRACE trace path
* CONNECT requires a tunneling protocol to connect to the proxy

### HTTP Persistent Connection

Keep the TCP connection state as long as either end does not explicitly propose to disconnect. Persistent Connection reduce TCP connect's duplicate build and disconnect. In HTTP/1.1, all connect default is persistent connection.

### Status control with Cookie

Cookie tech write Cookie info in Request ans Response body to control client status.  
Response has a header named *Set-Cookie* to tell client to save Cookie.

## HTTP information in HTTP message

HTTP message include two pieces, head and body, divided by (CR+LF).

### Message structure

message header include:

* request line or status line
* request/response header field
* general header field
* entity header field
* other

### Encoding to improve transmission effectiveness

message: HTTP communication basic unit, constituted with octet sequence.
entity: Request or response's payload.

common content encoding:

* gzip(GNU zip)
* compress
* deflate (zlib)
* identity (no encoding)

Chunked Transfer Coding chunk entity to multiple parts. Every part use hex to mark size.

### MIME type

Multipurpose Internet Mail Extensions

one message can include multiple types of entity *multipart/form-data* *multipart/byteranges*

### Range Request for part of content

user header *Range*

* -3000 before 3000
* 5000-7000 between 5000 and 7000
* 9000- after 9000
* -3000, 5000-7000 response with multipart/byteranges

### Content negotiation

headers:

* Accept
* Accept-Charset
* Accept-Encoding
* Accept-Language
* Content-Language

## Response's HTTP Status Code

* 1XX informational
* 2XX success
* 3XX redirection
* 4XX client error
* 5XX server error

common status code

1. 200 OK
2. 204 No Content (request successfully, but no resource)
3. 206 Partial Content (Range Request)
4. 301 Moved Permanently
5. 302 Found (Temporarily)
6. 303 See Other
7. 304 Not Modified (without entity, not redirection)
8. 307 Temporary Redirect
9. 400 Bad Request
10. 401 Unauthorized
11. 403 Forbidden
12. 404 Not Found
13. 500 Internal Server Error
14. 504 Service Unavailable

## Web server working with HTTP

### One virtual server implement multiple domains

header *Host* contains complete URI

### Proxy Gateway Tunnel

Proxy: forward client request, and add header *Via*.

Gateway: transform HTTP to other protocol.

Tunnel: security communication with SSL, TLS and so on.

### Cache resource

Cache: save resource copy in client or proxy server.

## HTTP Header

*header name* : *header value*

47 types of headers

General Header Fields:used by both request and response.

* Cache-Control
* Connection (control headers not forwarded or control persistent connection)
* Date
* Pragma
* Trailer (chunked transfer)
* Transfer-Encoding
* Upgrade
* Via
* Warning

Request Header Field

* Accept
* Accept-Charset
* Accept-Encoding
* Accept-Language
* Authorization
* Expect
* Form (email)
* Host
* If-Match
* If-Modified-Since
* If-None-Math
* If-Range
* If-Unmodified-Since
* Max-Forwards
* Proxy-Authorization
* Range
* Reference
* TE
* User-Agent

Response Header Field

* Accept-Ranges
* Age
* TRag
* Location
* Proxy-Authenticate
* Retry-After (work with 504 Service Unavailable or 3xx)
* Server
* Vary

Entity Header Fields

* Allow
* Content-Encoding
* Content-Language
* Content-Length
* Content-Location
* Content-MD5
* Content-Range
* Content-Type
* Expires
* Last-Modified

Cookie Headers

* Set-Cookie
* Cookie

Other Headers

* X-Frame-Options
* X-XSS-Protection
* DNT
* P3P

## HTTPS ensure HTTP security

C->S

Handshake:ClientHello

S->C

Handshake:ServerHello
Handshake:Certificate
Handshake:ServerHelloDone

C->S

Handshake:ClientKeyExchange
ChangeCipherSpec
Handshake:Finished

S->C

ChangeCipherSpec
Handshake:Finished

C->S

Application Data(HTTP)

S->C

Application Data(HTTP)

C->S

Alert:warning, close notify

## Authentication to confirm the identity of the accessing userConfirm user

### BASIC

userID:password base64 add header Authorization: Basic *hash*

### DIGEST

include realm, nonce, username, uri, and response

### SSL Client Authentication

use cookie to control session

## HTTP-Based Function Append Protocol

Ajax Asynchronous JavaScript and XML

SPDY speedy (session layer)

WebSocket

HTTP/2.0
WebDAV

## Construct Web Content Technology

HTML

CSS

Dynamic HTML

## Web Attack Technology

two types : active attack (e.g. SQL injection, OS injection), passive attack

Cross-Site Scripting, XSS

SQL Injection

OS Command Injection

HTTP Header Injection

Mail Header Injection

Directory Traversal

Remote File Inclusion

Forced Browsing

Open Redirect

Session Hijack

Session Fixation

Cross-Site Request Forgeries, CSRF

Password Cracking

Denial of Service attack (DoS)

Backdoor
