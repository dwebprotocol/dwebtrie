# hypertrie

Distributed single writer key/value store

```
npm install hypertrie
```

Uses a rolling hash array mapped trie to index key/value data on top of a [hypercore](https://github.com/mafintosh/hypercore).

Useful if you just want a straight forward single writer kv store or if you are looking for a building block for building more complex multiwriter databases on top.

## Usage

```js
const hypertrie = require('hypertrie')
const db = hypertrie('./trie.db', {valueEncoding: 'json'})

db.put('hello', 'world', function () {
  db.get('hello', console.log)
})
```

## API

#### `db = hypertrie(storage, [options])`

Create a new database. Options include:

```
{
  feed: aHypercore, // use this feed instead of loading storage
  valueEncoding: 'json' // set value encoding
}
```

If you set `options.feed` then you can set `storage` to null.

#### `db.get(key, callback)`

Lookup a key. Returns a result node if found or `null` otherwise.

#### `db.put(key, value, [callback])`

Insert a value

#### `db.del(key, [callback])`

Delete a key from the database.

#### `db.batch(batch, [callback])`

Insert/delete multiple values atomically.
The batch objects should look like this:

```js
{
  type: 'put' | 'del',
  key: 'key/we/are/updating',
  value: optionalValue
}
```

#### `const watcher = db.watch(prefix, [onchange])`

Watch a prefix of the db and get notified when it changes.

When there is a change `watcher.on('change')` is emitted.
Use `watcher.destroy()` to stop watching.

#### `version = db.version`

Returns the current version of the db (an incrementing integer).
Only available after `ready` has been emitted.

#### `checkoutDb = db.checkout(version)`

Returns a new db instance checked out at the version specified.

#### `checkoutDb = db.snapshot()`

Same as checkout but just returns the latest version as a checkout.

#### `stream = db.replicate([options])`

Returns a hypercore replication stream for the db. Pipe this together with another hypertrie instance.

All options are forwarded to hypercores replicate method.

#### `ite = db.iterator(prefix, [options])`

Returns a [nanoiterator](https://github.com/mafintosh/nanoiterator) that iterates
the latest values in the prefix specified.

Options include:

```js
{
  recursive: true
}
```

If you set `recursive: false` it will only iterate the immediate children (similar to readdir)

#### `stream = db.createReadStream(prefix, [options])`

Same as above but as a stream

#### `db.list(prefix, [options], callback)`

Creates an iterator for the prefix with the specified options and buffers it into an array that is passed to the callback.

#### `stream = db.createWriteStream()`

A writable stream you can write batch objects to, to update the db.

#### `ite = db.history([options])`

Returns a [nanoiterator](https://github.com/mafintosh/nanoiterator) that iterates over the feed in causal order.

Options include:

```js
{
  gt: seq,
  lt: seq,
  gte: seq,
  lte: seq,
  live: false // set to true to keep iterating forever
}
```

#### `stream = db.createHistoryStream([options])`

Same as above but as a stream

#### `ite = db.diff(version, [prefix])`

Returns a [nanoiterator](https://github.com/mafintosh/nanoiterator) that iterates the diff between the current db and the version you specifiy. The objects returned look like this

```js
{
  key: 'node-key-that-is-updated',
  left: <node>,
  right: <node>
}
```

If a node is in the current db but not in the version you are diffing against
`left` will be set to the current node and `right` will be null and vice versa.

#### `stream = db.createDiffStream(version, [prefix])`

Same as above but as a stream

## License

MIT
