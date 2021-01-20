# RESTful Specification

While all APIs implementing this
specification, aim to adhere as strictly
as possible to the RESTful standard,
elements of other API-Design standards
may have also been used.

## Resources

All APIs or packages,
implementing this specification,
MUST be based around the idea of
**resources**.

A resource is any named information,
that consistently adheres to a
specific schema.

All items of a resource MUST have
the same operations applicable to
them and support the same actions.

They MUST also consistently
support the same schema.  
However, operations MAY provide
only a varying subset of fields.

The names of all resources,
MUST be in plural.

**Examples:**

``/users/`` :heavy_check_mark:  
``/user/`` :x:

``/notes/`` :heavy_check_mark:  
``/note/`` :x:

Every item of a resource,
MUST have a self-identifying field,
of the following structure:  
``"self": {"href": "https://example.com/resource/id/"}``

If there is ever any ambiguity,
in whether a certain dataset
or nested data is a resource,
clients can check for the existance
of this field.

## Resource-Operations

The Drizm HATEOAS specification,
knows the following standard operations:

| HTTP-Method | Example-URL         | Canonical Name |
| :---------- | :------------------ | :------------- |
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
MAY also be implemented,
on a per-resource basis.

These will, by default, be applicable
only to singular items and use the
following URI-Style:  
``/resources/:id/action-name/``

Every resource MAY support a different
subset of operations and support *none*
or *n* different actions.

### Summary

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

#### Actions

**URI-Styles:**  
``/resources/:id/action-name/``

**HTTP-Methods:**  

- *POST*

## Response Types

Four main response types
are differentiated:

- Collections
- Items
- Errors
- Messages

### Collections

Produced by the **List** operation
of a resource.

``GET /images/`` -> Collection of Images

Collections MAY support both
**Pagination** and **Filtering**.  
However, this choice is collection
specific and does not have to be
uniform across a domain.

````json
[
  {
    "self": {"href":  "https://example.com/resources/1"}
  },
  {
    "self": {"href":  "https://example.com/resources/2"}
  }
]
````

### Items

Produced by the **Retrieve** operation
of a resource.

MAY also be returned by actions or
any other operations,
to signify the changes made
to an item by an update.

``GET /images/2/`` -> Singular image (item)  
``PATCH /images/2/`` -> Updated item

````json
{
  "self": {"href":  "https://example.com/images/2/"}
}
````

### Errors

These MUST be sent as
additional information,
accompanying 4xx and 5xx range
HTTP status-codes.

````json
{
  "code": "unknown_error",
  "detail": "An unknown error occured."
}
````

### Messages

These MAY be produced by actions.

````json
{
  "msg": "OK"
}
````

## Field Types

By default,
this spec will adhere to the
OpenAPI 3.0 specification of
types and their formats.

See: 
<a href="https://swagger.io/specification/" target="_blank">
https://swagger.io/specification/
</a>

### Hyperlinks

Hyperlinks here refers to an **absolute**
HTTP URI, that can be traversed,
without requiring any prior URL-building.

A hyperlink MUST support at least one
of the [standard operations](#resource-operations),
on exactly the provided URI.  
This is provided that the accessing user has
the required privileges to perform that operation.

Some generic textual fields may also contain
valid hyperlinks, however they will not be
marked as such and for the purposes of this
specification, will be considered plain-text.

All hyperlinks MUST be wrapped in a *"href"*
field, like so:  
``{href: "https://fully.qualified.uri/"}``

### Deferred Collections

A deferred collection, is a collection
that has not been loaded on the
current response.

The deferred collection MUST point
to a [hyperlink](#hyperlinks) of the
collection in question and specify the
size of the collection using the ``count``
attribute.

````json
{
  "self": {"href": "https://example.com/resources/2"},
  "some_collection": [
    {
      "count": 17,
      "href": "https://example.com/documents"
    }
  ]
}
````

A deferred collection MAY also signify
the abscence of a particular collection
in the *List* view of a resource.  
In that case, the collection will be
available and fully loaded on the
*Detail* view of that resource.  
This is specified by using an identical
*href* attribute to that of the resource
itself, but pointing to a specific field
using a fragment (#).

````json
[
    {
      "self": {"href": "https://example.com/resources/2"},
      "some_collection": [
        {
          "count": 17,
          "href": "https://example.com/resources/2#some_collection"
        }
      ]
    }
]
````

## Pagination

The LimitOffset Pagination style
will be used by all APIs implementing
this specification.

The parameters for the pagination,
must be sent as query params.

### Query Paramerers

#### limit

*required*

The maximum number of items to
return.  
If this is higher than the server
provided maximum, the server will
simply send a response,
containing the maximum number of
items.

#### offset

*optional*  
Default: 0

The offset from the first item
of the collection.  
If this is not provided,
the offset will simply be 0.

### Response Body Contents

#### count

The absolute number of items
of this resource,
that the client is allowed to query from.

#### next

A hyperlink to the next higher
offset of this paginated collection.

If no higher offset is available,
the value of this field 
will be ``null``.

#### prev

A hyperlink to the previous (lower)
offset of this paginated collection.

If no lower offset is available,
the value of this field 
will be ``null``.

#### results

The requested subset
of the paginated collection.

### Example

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
    {}
  ]
}
````

## Filtering

Querying is done through
the use of query params,
with the following structure:  
``/resources/?field_name=value``  
``/resources/?field_name!=value``  

These can be chained indefinetly.

Keywords are used to differentiate
between different lookup types.  
Not supplying any specific keyword
implies ``__exact``
(case-sensitive, exact) filtering.
