# JSON API Serializer
[![Build Status](https://travis-ci.org/SeyZ/jsonapi-serializer.svg?branch=master)](https://travis-ci.org/SeyZ/jsonapi-serializer)

A Node.js framework agnostic library for serializing your data to [JSON
API](http://jsonapi.org) (1.0 compliant).

## Installation
`$ npm install jsonapi-serializer`

## Documentation

**JSONAPISerializer(type, data, opts)** serializes the *data* (can be an object or an array) following the rules defined in *opts*.

- type: The resource type.
- data: An object to serialize.
- opts
    - *attributes*: An array of attributes to show. You can define an attribute as an option if you want to define some relationships (included or not).
        - *ref*: If present, it's considered as a [compount document](http://jsonapi.org/format/#document-compound-documents).
        - *attributes*: An array of attributes to show.
        - *topLevelLinks*: An object that describes the top-level links. Values can be *string* or a *function* (see examples below)
        - *dataLinks*: An object that describes the links inside data. Values can be *string* or a *function* (see examples below)
        - *relationshipLinks*: An object that describes the links inside relationships. Values can be *string* or a *function* (see examples below)


## Example

### Simple usage

```
var JSONAPISerializer = require('jsonapi-serializer');

new JSONAPISerializer('users', data, {
  topLevelLinks: { self: 'http://localhost:3000/api/users' },
  dataLinks: {
    self: function (user) {
      return 'http://localhost:3000/api/users/' + user.id
    }
  },
  attributes: ['firstName', 'lastName']
}).then(function (users) {
  // `users` here are JSON API compliant.
});
```

The result will be something like:

```
{
  "links": {
    "self": "http://localhost:3000/api/users"
  },
  "data": [{
    "type": "users",
    "id": "1",
    "attributes": {
      "first-name": "Sandro",
      "last-name": "Munda"
    },
    "links": "http://localhost:3000/api/users/1"
  }, {
    "type": "users",
    "id": "2",
    "attributes": {
      "first-name": "John",
      "last-name": "Doe"
    },
    "links": "http://localhost:3000/api/users/2"
  }]
}
```

### Nested resource
```
var JSONAPISerializer = require('jsonapi-serializer');

new JSONAPISerializer('users', data, {
  topLevelLinks: { self: 'http://localhost:3000/api/users' },
  attributes: ['firstName', 'lastName', 'address'],
  address: {
    attributes: ['addressLine1', 'zipCode', 'city']
  }
}).then(function (users) {
  // `users` here are JSON API compliant.
});
```

The result will be something like:

```
{
  "links": {
    "self": "http://localhost:3000/api/users"
  },
  "data": [{
    "type": "users",
    "id": "1",
    "attributes": {
      "first-name": "Sandro",
      "last-name": "Munda",
      "address": {
        "address-line1": "630 Central Avenue",
        "zip-code": 24012,
        "city": "Roanoke"
      }
    }
  }, {
    "type": "users",
    "id": "2",
    "attributes": {
      "first-name": "John",
      "last-name": "Doe",
      "address": {
        "address-line1": "400 State Street",
        "zip-code": 33702,
        "city": "Saint Petersburg"
      }
    }
  }]
}
```

### Compount document

```
var JSONAPISerializer = require('jsonapi-serializer');

new JSONAPISerializer('users', data, {
  topLevelLinks: { self: 'http://localhost:3000/api/users' },
  attributes: ['firstName', 'lastName', 'books'],
  books: {
    ref: '_id',
    attributes: ['title', 'isbn'],
    relationshipLinks: {
      "self": "http://example.com/relationships/books",
      "related": "http://example.com/books"
    },
    includedLinks: {
      self: 'http://example.com/relationships/author',
      related: function (book) {
        return 'http://example.com/books/' + book.id + '/author';
      }
    }
  }
}).then(function (users) {
  // `users` here are JSON API compliant.
});
```

The result will be something like:

```
{
  "links": {
    "self": "http://localhost:3000/api/users"
  },
  "data": [{
    "type": "users",
    "id": "1",
    "attributes": {
      "first-name": "Sandro",
      "last-name": "Munda"
    },
    "relationships": {
      "books": {
        "data": [
          { "type": "books", "id": "1" },
          { "type": "books", "id": "2" }
        ],
        "links": {
          "self": "http://example.com/relationships/books",
          "related": "http://example.com/books"
        }
      }
    }
  }, {
    "type": "users",
    "id": "2",
    "attributes": {
      "first-name": "John",
      "last-name": "Doe"
    },
    "relationships": {
      "books": {
        "data": [
          { "type": "books", "id": "3" }
        ],
        "links": {
          "self": "http://example.com/relationships/books",
          "related": "http://example.com/books"
        }
      }
    }
  }],
  "included": [{
  	"type": "books",
  	"id": "1",
  	"attributes": {
  	  "title": "La Vida Estilista",
  	  "isbn": "9992266589"
  	},
    "links": {
      self: 'http://example.com/relationships/author',
      related: 'http://example.com/books/1/author'
    }
  }, {
   "type": "books",
   "id": "2",
   "attributes": {
  	  "title": "La Maria Cebra",
  	  "isbn": "9992264446"
  	},
    "links": {
      self: 'http://example.com/relationships/author',
      related: 'http://example.com/books/2/author'
    }
  }, {
   "type": "books",
   "id": "3",
   "attributes": {
  	  "title": "El Salero Cangrejo",
  	  "isbn": "9992209739"
  	},
    "links": {
      self: 'http://example.com/relationships/author',
      related: 'http://example.com/books/3/author'
    }
  }]
}
```


# License

[MIT](https://github.com/SeyZ/jsonapi-serializer/blob/master/LICENSE)
