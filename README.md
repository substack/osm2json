# osm2json

[![Build Status](https://img.shields.io/travis/digidem/osm2json.svg)](https://travis-ci.org/digidem/osm2json)
[![npm](https://img.shields.io/npm/v/osm2json.svg)](https://www.npmjs.com/package/osm2json)
[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?maxAge=2592000)](http://standardjs.com/)

> Streaming parser from OSM XML to OSM JSON objects

Implements a [Node Transport Stream](http://nodejs.org/api/stream.html#stream_class_stream_transform). Takes a readable stream of [OSM XML](http://wiki.openstreetmap.org/wiki/OSM_XML) and outputs a stream of objects compatible with Overpass [OSM JSON](http://overpass-api.de/output_formats.html#json). Also reads [OsmChange](http://wiki.openstreetmap.org/wiki/OsmChange) XML and outputs the same format but with an additional property `action` which is one of `create`, `modify`, `delete`. Uses [sax-js](https://github.com/isaacs/sax-js) to work in both node and the browser.

NB: The name of this module is a bit of a misnomer - it outputs objects, not JSON.

## Table of Contents

- [Install](#install)
- [Usage](#usage)
- [API](#api)
- [Contribute](#contribute)
- [License](#license)

## Install

```
npm install osm2json
```

## Usage

```js
var fs = require('fs')
var Osm2Json = require('../lib/osm2json')

var rs = fs.createReadableStream(__dirname + './osm.xml')

rs.pipe(new Osm2Json()).pipe(process.stdout)
```

## Example Output

```js
// node
{
  type: 'node',
  id: 1,
  version: 0,
  timestamp: '2013-09-05T19:38:11.187Z',
  uid: 1,
  user: 'gregor',
  lat: 0,
  lon: 0,
  tags: { null: 'island' }
}

// way
{
  type: 'way',
  id: 3,
  version: 3,
  timestamp: '2013-09-05T19:38:11Z',
  changeset: 49,
  nodes: [ 19, 20, 21, 22, 26, 27 ],
  tags: { name: 'York St' }
}

// relation
{
  type: 'relation',
  id: 1,
  members: [
    {
      type: 'relation',
      ref: 1745069,
      role: 'outer'
    },
    {
      type: 'relation',
      ref: 172789
    }
  ],
  tags: {
    from: 'Konrad-Adenauer-Platz',
    name: 'VRS 636'
  }
}
```

## Example: Actual JSON output

```js
var through = require('through2')
var fs = require('fs')
var path = require('path')
var Osm2Json = require('../lib/osm2json')

var rs = fs.createReadStream(path.join(__dirname, '../test/osm.xml'))

var jsonStream = through.obj(write, end)

jsonStream.push('[')
var start = true

rs.pipe(new Osm2Json()).pipe(jsonStream).pipe(process.stdout)

function write (row, enc, next) {
  if (!start) {
    this.push(', ')
  } else {
    start = false
  }
  next(null, JSON.stringify(row, null, 2))
}

function end (next) {
  this.push(']\n')
}
```

## API

```js
var Osm2Json = require('osm2json')
```

### var stream = new Osm2Json(opts)

Create a transform stream with:

* `opts.coerceIds` - coerce id-type fields (`id, uid, version, changeset, ref`) to `Number` (default `true`) - useful for [osm-p2p-db](https://github.com/digidem/osm-p2p-db) where ids can be strings.
* `opts.bounds` - Also parse bounds (default `true`)
* `opts.types` - An array of element types you are interested in, e.g. `opts.types = ['node']` (default `['node', 'way', 'relation', 'changeset']`)
* `opts.strict` - Be a jerk about XML (default `false`). In strict mode will throw an error if:
  - XML is badly formatted
  - Case of element names differs from spec
  - Root node is not one of `osm`, `osmChange`, `diffResult`
  - An action element (`create, modify, delete`) appears when the root is not `osmChange`
  - Any element in the XML which is not one of `create, modify, delete, node, way, relation, changeset, bounds, nd, tag, member`

Any attribute that is not a valid OSM XML attribute will be ignored (see [`WHITELISTS`](https://github.com/digidem/osm2json/blob/master/lib/osm2json.js#L27-L48)). `tag`, `member`, or `nd` elements without the required attributes will throw an error. The readable side of the stream is in `objectMode`.

### stream.parse(str)

Parse `str` and return the result. Will throw any error.

## Contribute

PRs welcome. Please follow [JS Standard Style](http://standardjs.com/). Right now this could do with some tests. If you are feeling ambitious, this could be sped up by using [node-expat](https://github.com/astro/node-expat) on node. The interface is similar to sax-js and it should be possible to wrap this to use sax-js on the browser and node-expat on the server using the [browserify `browser` field](https://github.com/substack/browserify-handbook#browser-field)

## License

MIT (c) 2016, Digital Democracy.
