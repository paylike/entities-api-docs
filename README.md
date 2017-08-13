# Pawns API

- [Basics](#basics)
- [Identities](#identities)
	- [Create an identity](#create-an-identity)
- [Privileges](#privileges)
	- [Create a privilege](#create-an-privilege)
- [Entities](#entities)
	- [Create an entity](#create-an-entity)
	- [Fetch an entity](#fetch-an-entity)
- [Residences](#residences)
	- [Create a residence](#create-a-residence)
	- [Fetch a residence](#fetch-a-residence)
	- [Search residences](#search-residences)
- [Links](#links)
	- [Create a link](#create-a-link)
	- [Search links](#search-links)
- [Relationships](#relationships)
	- [Create a relationship](#create-upload-a-relationships)
	- [Search relationships](#search-relationships)
- [Documents](#documents)
	- [Create a document](#create-a-document)
	- [Fetch a document](#fetch-a-document)
	- [Search documents](#search-documents)
- [Files](#files)
	- [Create (upload) a file](#create-upload-a-file)
	- [Fetch (download) a file](#fetch-download-a-file)

## Basics

This API is exposed using the HTTP protocol.

### Request

Any response body must be either `JSON` encoded with a `Content-Type` header
set to `application/json` or binary with a corresponding `Content-Type` (e.g.
for file uploads).

#### Authentication and authorization

Most endpoints require you to authenticate. [Create an identity](#create-an-
identity) to get an API key.

Supply an API key as the username part of a HTTP basic authentication.

```shell
Authorization: Basic $(echo $API_KEY | base64)
```

Using cURL:

```shell
curl -i -u $API_KEY https://pawns/entities/ping
```

### Response

All successful responses will have status codes in the 200-299 range.

The content type, if data is returned, will be `application/x-ndjson;
charset=utf-8`. [NDJSON](http://ndjson.org/) simply means JSON but with top
level arrays delimited by newlines.

### Create

Use the HTTP `POST` method to submit data.

```shell
curl -i https://pawns/entities \
	-H 'Content-Type: application/json' \
	-d <data>
```

A successful request will return a `201` status code and data with a format
of:

```js
{
	id: String,
}
```

### Fetch

Use the HTTP `GET` method to fetch data.

```shell
curl -i https://pawns/entities/$ID \
	-H 'Accept-Type: application/x-ndjson'
```

A successful request will return a `200` status code.

## Identities

### Create an identity

`/identities`

This endpoint accepts no data.

This endpoint requires no authentication.

Returns:

```js
{
	id: String,
	key: String,
}
```

## Privileges

### Create a privilege

So far only a single type of privilege exists. Thus having a privilege
implicitly means read and write access to an entity.

`/privileges`

```js
{
	identityId: String,
	entityId: String,
}
```

To create a new privilege, the authenticated identity must itself have a
privilege for the same entity.

## Entities

### Create an entity

`/entities`

Set one off `individual` or `corporation` to `true`.

```js
{
	individual: Boolean,
	corporation: Boolean,

	name: String,			// legal name

	// only if "individual" is "true"
	birth: {
		date: {
			year: Number,	// four digits year
			month: Number,	// month (1-12)
			day: Number,	// day (1-31)
		},
		country: String,	// ISO 3166-1 alpha2 code
	},

	// only if "corporation" is "true"
	country: String,		// ISO 3166-1 alpha2 code, Country of incorporation
}
```

Creating an entity automatically adds a privilege of the creating identity and
the new entity.

### Fetch an entity

`/entities/:id`

## Residences

### Create a residence

`/residences`

```js
{
	entityId: String,
	domicile: Boolean,	// headquarter, primary address
	address: {
		// yet to be defined
	},
	country: String,	// ISO 3166-1 alpha2 code
}
```

### Fetch a residence

`/residences/:id`

### Search residences

```
/residences?entityId=:entityId
```

`entityId` is required.

## Links

Provides a way of linking an entity to external registries such as business
registries or social security systems.

### Create a link

`/links`

```js
{
	entityId: String,
	system: {
		id: String,		// identifier of the external system
		key: String,	// primary key in the external system
	}
}
```

### Search links

`/links?entityId=:entityId`

`entityId` is required.

## Relationships

Relationships are way to specify how two entities may be related.

### Create a relationship

```js
{
	entities: Array,	// array of entityIds
	type: String,
}
```

An example `type` would be `owner` in which case `entities[0]` would be an
owner of `entities[1]`.

### Search relationships

`/links?entityId=:entityId`

`entityId` is required.

## Documents

### Create a document

`/documents`

```js
{
	entityId: String,
	fileId: uuid.parse(body.fileId),

	// optional
	hasPhoto: Boolean,
	hasAddress: Boolean,
	hasName: Boolean,

	// specifies that this document is related somehow to a residence, it
	// might be a "proof of address".
	residenceId: String,
}
```

### Fetch a document

`/documents/:id`

### Search documents

```
/documents?
	entityId
	residenceId
```

`entityId` is required.

## Files

### Create (upload) a file

`/files`

Requires a `Content-Type` HTTP header with the media/mime type of the file
(e.g. image/jpeg).

Optional HTTP headers:

```
X-Filename		# Name of file
```

*Expects the request body to be the binary data*

### Fetch (download) a file

`/files/:id`
