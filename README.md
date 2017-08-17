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

Error codes:

```
400 Validation error
401 Missing or unknown API key
403 Forbidden (API key OK, but no access)
404 Unknown URL
```

### Create

Use the HTTP `POST` method to submit data.

```shell
curl -i https://pawns/entities \
	-H 'Content-Type: application/json' \
	-d <data>
```

Attributes marked with `*` in these docs are required.

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

Most responses will include an `id` which is system-wide unique, and a
`creatorId` referencing the identity who added the data. The date of creation
can be extracted from the ID ([see this example](https://github.com/srcagency/date-from-object-id)).

## Identities

### Create an identity

`/identities`

This endpoint accepts no data.

This endpoint requires no authentication.

Returns:

```js
{
	id: String,
	creatorId: String,
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
	identityId: String*,	// existing identity ID
	entityId: String*,		// existing entity ID
}
```

To create a new privilege, the authenticated identity must itself have a
privilege for the same entity.

## Entities

### Create an entity

`/entities`

Set one of `individual` or `corporation` to `true`.

```js
{
	// mutually exclusive
	individual: Boolean,
	corporation: Boolean,

	name: String,			// legal name (max length: 256)

	// registered birth or official incorporation
	birth: {
		// optional as a whole
		date: {
			year: Number*,	// four digits year
			month: Number*,	// month (1-12)
			day: Number*,	// day (1-31)
		},

		country: String,	// ISO 3166-1 alpha2 code
	},
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
	entityId: String*,	// existing entity ID
	domicile: Boolean,	// headquarter, primary address
	address: String,	// official address (max length: 512)
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
	entityId: String*,	// existing entity ID
	system: {
		id: String*,	// identifier of the external system (length: 1-64)
		key: String*,	// primary key in the external system (length: 1-128)
	},
}
```

### Search links

`/links?entityId=:entityId`

`entityId` is required.

## Relationships

Relationships are a way to describe how two entities are related.

### Create a relationship

```js
{
	entities: Array*,	// array of existing entity IDs (length: 2)
	type: String*,		// nature of relationship (length: 1-64)
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
	entityId: String*,		// existing entity ID
	fileId: String*,		// existing file ID (UUID)

	// optional
	tags: Array(String),	// max length: 16, strings length: 1-64

	// specifies that this document is related somehow to a residence, it
	// might be a "proof of address".
	residenceId: String,	// existing residence ID
}
```

### Fetch a document

`/documents/:id`

Returns:

```js
{
	id,
	creatorId,
	created,
	entityId,
	residenceId,
	tags,

	file: {
		id,
		type,
		name,
	},
}
```

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
