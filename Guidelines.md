## 1. Consistency fundamentals
### 1.1. URL structure
Humans SHOULD be able to easily read and construct URLs.

This facilitates discovery and eases adoption on platforms without a well-supported client library.

An example of a well-structured URL is:

```
https://api.anvui.vn/v1.0/people/cto@anvui.vn/inbox
```

An example URL that is not friendly is:

```
https://api.anvui.vn/EWS/OData/Users('cto@anvui.vn.com')/Folders('AAMkAadaDdiYzI1MjUzLTk4MjQtNDQ1Yy05YjJkLWNlMzMzYmIzNTY0MwAuAAAAAACzMsPHYH6HQoSwfdpDx-2bAQCXhUk6PC1dS7AERFluCgBfAAABo58UAAA=')
```

A frequent pattern that comes up is the use of URLs as values.
Services MAY use URLs as values.
For example, the following is acceptable:

```
https://api.anvui.vn/v1.0/items?url=https://resources.anvui.vn/shoes/fancy
```

### 1.2. URL length
The HTTP 1.1 message format, defined in RFC 7230, in section [3.1.1][rfc-7230-3-1-1], defines no length limit on the Request Line, which includes the target URL.
From the RFC:

> HTTP does not place a predefined limit on the length of a
   request-line. [...] A server that receives a request-target longer than any URI it wishes to parse MUST respond
   with a 414 (URI Too Long) status code.

Services that can generate URLs longer than 2,083 characters MUST make accommodations for the clients they wish to support.
Here are some sources for determining what target clients support:

 * [http://stackoverflow.com/a/417184](http://stackoverflow.com/a/417184)
 
Also note that some technology stacks have hard and adjustable URL limits, so keep this in mind as you design your services.

### 1.3. Canonical identifier
In addition to friendly URLs, resources that can be moved or be renamed SHOULD expose a URL that contains a unique stable identifier.
It MAY be necessary to interact with the service to obtain a stable URL from the friendly name for the resource, as in the case of the "/my" shortcut used by some services.

The stable identifier is not required to be a GUID.

An example of a URL containing a canonical identifier is:

```
https://api.anvui.vn/v1.0/people/7011042402/inbox
```

### 1.4. Supported methods
Operations MUST use the proper HTTP methods whenever possible, and operation idempotency MUST be respected.
HTTP methods are frequently referred to as the HTTP verbs.
The terms are synonymous in this context, however the HTTP specification uses the term method.

Below is a list of methods that An Vui REST services SHOULD support.
Not all resources will support all methods, but all resources using the methods below MUST conform to their usage.

Method  | Description                                                                                                                | Is Idempotent
------- | -------------------------------------------------------------------------------------------------------------------------- | -------------
GET     | Return the current value of an object   or list sub-objects                                                                | True
PUT     | Replace an object, or create a named object, when applicable                                                               | True
DELETE  | Delete an object                                                                                                           | True
POST    | Create a new object based on the data provided, or submit a command                                                        | False
HEAD    | Return metadata of an object for a GET response. Resources that support the GET method MAY support the HEAD method as well | True
PATCH   | Apply a partial update to an object                                                                                        | False
OPTIONS | Get information about a request; see below for details.                                                                    | True

<small>Table 1</small>

#### 1.4.1. POST
POST operations SHOULD support the Location response header to specify the location of any created resource that was not explicitly named, via the Location header.

As an example, imagine a service that allows creation of hosted servers, which will be named by the service:

```http
POST http://api.anvui.vn/account1/servers
```

The response would be something like:

```http
201 Created
Location: http://api.anvui.vn/account1/servers/server321
```

Where "server321" is the service-allocated server name.

Services MAY also return the full metadata for the created item in the response.

#### 1.4.2. PATCH
PATCH has been standardized by IETF as the method to be used for updating an existing object incrementally (see [RFC 5789][rfc-5789]).
Anvui REST API Guidelines compliant APIs SHOULD support PATCH.

#### 1.4.3. Creating resources via PATCH (UPSERT semantics)
Services that allow callers to specify key values on create SHOULD support UPSERT semantics, and those that do MUST support creating resources using PATCH.
Because PUT is defined as a complete replacement of the content, it is dangerous for clients to use PUT to modify data.
Clients that do not understand (and hence ignore) properties on a resource are not likely to provide them on a PUT when trying to update a resource, hence such properties could be inadvertently removed.
Services MAY optionally support PUT to update existing resources, but if they do they MUST use replacement semantics (that is, after the PUT, the resource's properties MUST match what was provided in the request, including deleting any server properties that were not provided).

Under UPSERT semantics, a PATCH call to a nonexistent resource is handled by the server as a "create," and a PATCH call to an existing resource is handled as an "update." To ensure that an update request is not treated as a create or vice versa, the client MAY specify precondition HTTP headers in the request.
The service MUST NOT treat a PATCH request as an insert if it contains an If-Match header and MUST NOT treat a PATCH request as an update if it contains an If-None-Match header with a value of "*".

If a service does not support UPSERT, then a PATCH call against a resource that does not exist MUST result in an HTTP "409 Conflict" error.

#### 1.4.4. Options and link headers
OPTIONS allows a client to retrieve information about a resource, at a minimum by returning the Allow header denoting the valid methods for this resource.

In addition, services SHOULD include a Link header (see [RFC 5988][rfc-5988]) to point to documentation for the resource in question:

```http
Link: <{help}>; rel="help"
```

Where {help} is the URL to a documentation resource.

For examples on use of OPTIONS, see [preflighting CORS cross-domain calls][cors-preflight].

### 1.5. Standard request headers
The table of request headers below SHOULD be used by Anvui REST API Guidelines services.
Using these headers is not mandated, but if used they MUST be used consistently.

All header values MUST follow the syntax rules set forth in the specification where the header field is defined.
Many HTTP headers are defined in [RFC7231][rfc-7231], however a complete list of approved headers can be found in the [IANA Header Registry][IANA-headers]."

Header                            | Type                                  | Description
--------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Authorization                     | String                                           | Authorization header for the request
Date                              | Date                                             | Timestamp of the request, based on the client's clock, in [RFC 5322][rfc-5322-3-3] date and time format.  The server SHOULD NOT make any assumptions about the accuracy of the client's clock.  This header MAY be included in the request, but MUST be in this format when supplied.  Greenwich Mean Time (GMT) MUST be used as the time zone reference for this header when it is provided.  For example: `Wed, 24 Aug 2016 18:41:30 GMT`.  Note that GMT is exactly equal to UTC (Coordinated Universal Time) for this purpose.
Accept                            | Content type                                     | The requested content type for the response such as: <ul><li>application/xml</li><li>text/xml</li><li>application/json</li><li>text/javascript (for JSONP)</li></ul>Per the HTTP guidelines, this is just a hint and responses MAY have a different content type, such as a blob fetch where a successful response will just be the blob stream as the payload. For services following OData, the preference order specified in OData SHOULD be followed.
Accept-Encoding                   | Gzip, deflate                                    | REST endpoints SHOULD support GZIP and DEFLATE encoding, when applicable. For very large resources, services MAY ignore and return uncompressed data.
Accept-Language                   | "en", "es", etc.                                 | Specifies the preferred language for the response. Services are not required to support this, but if a service supports localization it MUST do so through the Accept-Language header.
Accept-Charset                    | Charset type like "UTF-8"                        | Default is UTF-8, but services SHOULD be able to handle ISO-8859-1.
Content-Type                      | Content type                                     | Mime type of request body (PUT/POST/PATCH)
Prefer                            | return=minimal, return=representation            | If the return=minimal preference is specified, services SHOULD return an empty body in response to a successful insert or update. If return=representation is specified, services SHOULD return the created or updated resource in the response. Services SHOULD support this header if they have scenarios where clients would sometimes benefit from responses, but sometimes the response would impose too much of a hit on bandwidth.
If-Match, If-None-Match, If-Range | String                                           | Services that support updates to resources using optimistic concurrency control MUST support the If-Match header to do so. Services MAY also use other headers related to ETags as long as they follow the HTTP specification.

### 1.6. Standard response headers
Services SHOULD return the following response headers, except where noted in the "required" column.

Response Header    | Required                                      | Description
------------------ | --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Date               | All responses                                 | Timestamp the response was processed, based on the server's clock, in [RFC 5322][rfc-5322-3-3] date and time format.  This header MUST be included in the response.  Greenwich Mean Time (GMT) MUST be used as the time zone reference for this header.  For example: `Wed, 24 Aug 2016 18:41:30 GMT`. Note that GMT is exactly equal to UTC (Coordinated Universal Time) for this purpose.
Content-Type       | All responses                                 | The content type
Content-Encoding   | All responses                                 | GZIP or DEFLATE, as appropriate
Preference-Applied | When specified in request                     | Whether a preference indicated in the Prefer request header was applied
ETag               | When the requested resource has an entity tag | The ETag response-header field provides the current value of the entity tag for the requested variant. Used with If-Match, If-None-Match and If-Range to implement optimistic concurrency control.

### 1.7. Custom headers
Custom headers MUST NOT be required for the basic operation of a given API.

Some of the guidelines in this document prescribe the use of nonstandard HTTP headers.
In addition, some services MAY need to add extra functionality, which is exposed via HTTP headers.
The following guidelines help maintain consistency across usage of custom headers.

Headers that are not standard HTTP headers MUST have one of two formats:

1. A generic format for headers that are registered as "provisional" with IANA ([RFC 3864][rfc-3864])
2. A scoped format for headers that are too usage-specific for registration

These two formats are described below.

### 1.8. Specifying headers as query parameters
Some headers pose challenges for some scenarios such as AJAX clients, especially when making cross-domain calls where adding headers MAY not be supported.
As such, some headers MAY be accepted as Query Parameters in addition to headers, with the same naming as the header:

Not all headers make sense as query parameters, including most standard HTTP headers.

The criteria for considering when to accept headers as parameters are:

1. Any custom headers MUST be also accepted as parameters.
2. Required standard headers MAY be accepted as parameters.
3. Required headers with security sensitivity (e.g., Authorization header) MIGHT NOT be appropriate as parameters; the service owner SHOULD evaluate these on a case-by-case basis.

The one exception to this rule is the Accept header.
It's common practice to use a scheme with simple names instead of the full functionality described in the HTTP specification for Accept.

### 1.9. PII parameters
Consistent with their organization's privacy policy, clients SHOULD NOT transmit personally identifiable information (PII) parameters in the URL (as part of path or query string) because this information can be inadvertently exposed via client, network, and server logs and other mechanisms.

Consequently, a service SHOULD accept PII parameters transmitted as headers.

However, there are many scenarios where the above recommendations cannot be followed due to client or software limitations.
To address these limitations, services SHOULD also accept these PII parameters as part of the URL consistent with the rest of these guidelines.

Services that accept PII parameters -- whether in the URL or as headers -- SHOULD be compliant with privacy policy specified by their organization's engineering leadership.
This will typically include recommending that clients prefer headers for transmission and implementations adhere to special precautions to ensure that logs and other service data collection are properly handled.

### 1.10. Response formats
For organizations to have a successful platform, they must serve data in formats developers are accustomed to using, and in consistent ways that allow developers to handle responses with common code.

Web-based communication, especially when a mobile or other low-bandwidth client is involved, has moved quickly in the direction of JSON for a variety of reasons, including its tendency to be lighter weight and its ease of consumption with JavaScript-based clients.

JSON property names SHOULD be camelCased.

Services SHOULD provide JSON as the default encoding.

#### 1.10.1. Clients-specified response format
In HTTP, response format SHOULD be requested by the client using the Accept header.
This is a hint, and the server MAY ignore it if it chooses to, even if this isn't typical of well-behaved servers.
Clients MAY send multiple Accept headers and the service MAY choose one of them.

The default response format (no Accept header provided) SHOULD be application/json, and all services MUST support application/json.

Accept Header    | Response type                      | Notes
---------------- | ---------------------------------- | -------------------------------------------
application/json | Payload SHOULD be returned as JSON | Also accept text/javascript for JSONP cases

```http
GET https://api.anvui.vn/v1.0/products/user
Accept: application/json
```
