# Drizm API Spec

This repository contains a formal documentation
of the Drizm HATEOAS specification.

When referring to the term "HATEOAS" in this
document, the authors are referring to a
Hypermedia application language for JSON based
RESTful or GraphQL Web-APIs.

As the currently used format is REST,
a section for GraphQL is not inclued in this
document.

## Implementations

**Python - Django:**  
<a href="https://github.com/drizm-team/django-commons" target="_blank">
drizm-django-commons
</a>

**Typescript - Angular:**  
<a href="https://github.com/drizm-team/angular-template" target="_blank">
Drizm Angular Template - RestService
</a>

## RESTful Specification

While all APIs implementing this
specification, aim to adhere as strictly
as possible to the RESTful standard,
elements of other API-Design standards
may also be used.

### Resources

All APIs following this specification,
will be based around the idea of
**resources**.

A resource is any named information,
that consistently adheres to a
specific schema.

All items of a resource will have
the same operations applicable to
them and support the same actions.

They will also consistently
support the same schema, however,
operations may have a varying subset
of fields.

The names of all resources,
will be in plural.

**Examples:**  
``/users/`` :heavy_check_mark:  
``/user/`` :x:  

``/tokens/`` :heavy_check_mark:  
``/token/`` :x:

Every item of a resource,
must have a self-identifying field,
of the following structure:  
``"self": {"href": "https://example.com/resource/id/"}``

If there is ever any ambiguity,
in whether a certain dataset
or nested data is a resource,
clients can check for the existance
of this field.

### Operations

The Drizm HATEOAS specification,
by default, knows the following standard
operations:

| HTTP-Method | Example             | Canonical Name |
| :---------: | :-----------------: | :------------: |
| *GET*       | ``/resources/``     | List           |
| *GET*       | ``/resources/:id/`` | Retrieve       |
| *POST*      | ``/resources/``     | Create         |
| *PUT*       | ``/resources/:id/`` | Upsert         |
| *PUT*       | ``/resources/:id/`` | Replace        |
| *PATCH*     | ``/resources/:id/`` | Update         |
| *DELETE*    | ``/resources/:id/`` | Destroy        |

The following methods are also implictly
supported for all resource endpoints:

- *HEAD*
- *OPTIONS*

In addition to the above operations,
non-standard RPC-style **actions**
may also be implemented,
on a per-resource basis.

These will, by default, be applicable
only to singular items and use the
following URI-Style:  
``/resources/:id/action-name/``

Every resource may support a different
subset of operations and support *none*
or *n* different actions.

#### Summary

#### Operations  

**URI-Styles:**  
``/resources/``  
``/resources/:id/``  

**HTTP-Methods:**  

- *GET*
- *POST*
- *PATCH*
- *PUT*
- *DELETE*

##### Actions

**URI-Styles:**  
``/resources/:id/action-name/``

**HTTP-Methods:**  

- *POST*

### Response Types

Four response types are differentiated:

- Collections
- Items
- Errors
- Messages

#### Collections

Produced by the **List** operation
of a resource.

``GET /images/`` -> Collection of Images

Collections may support both
**Pagination** and **Filtering**,
however this is on a per-resource
basis.

##### Schema

````json
[
  {
    "self": {"href":  "https://example.com/resources/1"},
    ...
  },
  {
    "self": {"href":  "https://example.com/resources/2"},
    ...
  }
]
````

#### Items

Produced by the **Retrieve** operation
of a resource.

May also be returned by actions or
any other operations,
to signify the changes made
to an item by an update.

``GET /images/2/`` -> Singular image (item)
``PATCH /images/2/`` -> Updated item

##### Schema

````json
{
  "self": {"href":  "https://example.com/images/2/"},
  ...
}
````

#### Errors

These will be sent as
additional information,
accompanying 4xx and 5xx range
HTTP status-codes.

##### Schema

````json
{
  "code": "unknown_error",
  "detail": "An unknown error occured."
}
````

#### Messages

These may be produced by actions.

##### Schema

````json
{
  "msg": "OK"
}
````

### Field Types

By default,
this spec will adhere to the
OpenAPI 3.0 specification of
types and their formats.

See: 
<a href="https://swagger.io/specification/" target="_blank">
https://swagger.io/specification/
</a>

#### Hyperlinks

Hyperlinks here refers to an **absolute**
HTTP URI, that can be traversed,
without requiring any prior URL-building.

A *GET* of a hyperlink, should always
return information if you have the
necessary permissions.

Some generic textual fields may also contain
hyperlinks, however they will not be marked
as such and for the purposes of this
specification, will be considered plain-text.

All hyperlinks will be wrapped in a "href"
field, like so:  
``{href: "https://fully.qualified.uri/"}``

### Pagination

The LimitOffset Pagination style
will be used by all APIs implementing
this specification.

The parameters for the pagination,
must be sent as query params.

#### Query Paramerers

##### limit

*required*

The maximum number of items to
return.  
If this is higher than the server
provided maximum, the server will
simply send a response,
containing the maximum number of
items.

##### offset

*optional*  
Default: 0

The offset from the first item
of the collection.  
If this is not provided,
the offset will simply be 0.

#### Response Body Contents

##### count

The absolute number of items
of this resource,
that the client is allowed to query from.

##### next

A hyperlink to the next higher
offset of this paginated collection.

If no higher offset is available,
the value of this field 
will be ``null``.

##### prev

A hyperlink to the previous (lower)
offset of this paginated collection.

If no lower offset is available,
the value of this field 
will be ``null``.

##### results

The requested subset
of the paginated collection.

#### Example

**Request:**  
``GET https://example.com/resources/?limit=100&offset=400``

**Response:**  
``HTTP 200``

````json
{
  "count": 1023,
  "next": {
    "href": "https://example.com/resources/?limit=100&offset=500"
  },
  "prev": {
    "href": "https://example.com/resources/?limit=100&offset=300"
  },
  "results": [
    ...
  ]
}
````

### Filtering

Querying is done through
the use of query params,
with the following structure:  
``/resources/?field_name=value``  
``/resources/?field_name!=value``  

These can be chained indefinetly.

Keywords are used to differentiate
between different lookup types.  
Not supplying a specific keyword
implies ``__exact`` filtering.
