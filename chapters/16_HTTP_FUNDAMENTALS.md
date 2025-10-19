# Chapter 16: HTTP Fundamentals Crash Course

## 16.1 HTTP vs HTTPS

**HTTP:** Unsecured (data visible)
**HTTPS:** Secured with SSL/TLS encryption

### Enable HTTPS in Production

All modern apps should use HTTPS!

## 16.2 URI, URL, and URN

```
URI: http://example.com/api/users?id=123#section
 ├── URL: Locator (where to find it)
 └── URN: Name (what it is)
```

## 16.3 HTTP Headers

### Request Headers
```
GET /api/users HTTP/1.1
Host: example.com
Authorization: Bearer token123
Content-Type: application/json
User-Agent: Mozilla/5.0
Accept: application/json
```

### Response Headers
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 256
Set-Cookie: sessionId=abc123
```

## 16.4 Common HTTP Headers

```
Accept: application/json
User-Agent: Mozilla/5.0
Authorization: Bearer token
Content-Type: application/json
Cache-Control: no-cache
X-Custom-Header: value
```

## 16.5 HTTP Methods

```
GET     - Retrieve data
POST    - Create data
PUT     - Replace entirely
PATCH   - Update partially
DELETE  - Remove data
HEAD    - Like GET, no body
OPTIONS - What methods allowed
```

## 16.6 HTTP Status Codes

```
2xx Success:
200 OK
201 Created
204 No Content

3xx Redirect:
301 Moved
304 Not Modified

4xx Client Error:
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found

5xx Server Error:
500 Internal Error
503 Unavailable
```

---

## 🎯 Next: Chapter 17 - File Upload Implementation
