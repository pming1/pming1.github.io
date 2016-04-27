---
title: multer
description: Middleware for handling <code>multipart/form-data</code>.
author: Feynman
date: 2016-04-27 17:26:54
categories: nodejs
tags: nodejs upload multer
---

### 中间件名称：multer
### 用途：Middleware for handling <code>multipart/form-data</code>.
### 源码地址：[github/multer](https://github.com/expressjs/multer)

----

# 安装
` npm install --save multer `

# 使用说明

Multer adds a body object and a file or files object to the request object. The body object contains the values of the text fields of the form, the file or files object contains the files uploaded via the form.

## 简单例子：

```
var express = require('express')
var multer  = require('multer')
var upload = multer({ dest: 'uploads/' })

var app = express()

app.post('/profile', upload.single('avatar'), function (req, res, next) {
  // req.file is the `avatar` file
  // req.body will hold the text fields, if there were any
})

app.post('/photos/upload', upload.array('photos', 12), function (req, res, next) {
  // req.files is array of `photos` files
  // req.body will contain the text fields, if there were any
})

var cpUpload = upload.fields([{ name: 'avatar', maxCount: 1 }, { name: 'gallery', maxCount: 8 }])
app.post('/cool-profile', cpUpload, function (req, res, next) {
  // req.files is an object (String -> Array) where fieldname is the key, and the value is array of files
  //
  // e.g.
  //  req.files['avatar'][0] -> File
  //  req.files['gallery'] -> Array
  //
  // req.body will contain the text fields, if there were any
})
```

In case you need to handle a text-only multipart form, you can use any of the multer methods (.single(), .array(), fields()). Here is an example using .array():
```
var express = require('express')
var app = express()
var multer  = require('multer')
var upload = multer()

app.post('/profile', upload.array(), function (req, res, next) {
  // req.body contains the text fields
})
```

# API
## File information
### Each file contains the following information:
Key | Description | Note
--- | --- | ---
`fieldname` | Field name specified in the form |
`originalname` | Name of the file on the user's computer |
`encoding` | Encoding type of the file |
`mimetype` | Mime type of the file |
`size` | Size of the file in bytes |
`destination` | The folder to which the file has been saved | `DiskStorage`
`filename` | The name of the file within the `destination` | `DiskStorage`
`path` | The full path to the uploaded file | `DiskStorage`
`buffer` | A `Buffer` of the entire file | `MemoryStorage`

### `multer(opts)`

Multer accepts an options object, the most basic of which is the `dest`
property, which tells Multer where to upload the files. In case you omit the
options object, the files will be kept in memory and never written to disk.

By default, Multer will rename the files so as to avoid naming conflicts. The
renaming function can be customized according to your needs.

The following are the options that can be passed to Multer.

Key | Description
--- | ---
`dest` or `storage` | Where to store the files
`fileFilter` | Function to control which files are accepted
`limits` | Limits of the uploaded data

In an average web app, only `dest` might be required, and configured as shown in
the following example.

```javascript
var upload = multer({ dest: 'uploads/' })
```

If you want more control over your uploads, you'll want to use the `storage`
option instead of `dest`. Multer ships with storage engines `DiskStorage`
and `MemoryStorage`; More engines are available from third parties.

#### `.single(fieldname)`

Accept a single file with the name `fieldname`. The single file will be stored
in `req.file`.

#### `.array(fieldname[, maxCount])`

Accept an array of files, all with the name `fieldname`. Optionally error out if
more than `maxCount` files are uploaded. The array of files will be stored in
`req.files`.

#### `.fields(fields)`

Accept a mix of files, specified by `fields`. An object with arrays of files
will be stored in `req.files`.

`fields` should be an array of objects with `name` and optionally a `maxCount`.
Example:

```javascript
[
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]
```

#### `.any()`

Accepts all files that comes over the wire. An array of files will be stored in
`req.files`.

**WARNING:** Make sure that you always handle the files that a user uploads.
Never add multer as a global middleware since a malicious user could upload
files to a route that you didn't anticipate. Only use this function on routes
where you are handling the uploaded files.

### `storage`

#### `DiskStorage`

The disk storage engine gives you full control on storing files to disk.

```javascript
var storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, '/tmp/my-uploads')
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now())
  }
})

var upload = multer({ storage: storage })
```

There are two options available, `destination` and `filename`. They are both
functions that determine where the file should be stored.

`destination` is used to determine within which folder the uploaded files should
be stored. This can also be given as a `string` (e.g. `'/tmp/uploads'`). If no
`destination` is given, the operating system's default directory for temporary
files is used.

**Note:** You are responsible for creating the directory when providing
`destination` as a function. When passing a string, multer will make sure that
the directory is created for you.

`filename` is used to determine what the file should be named inside the folder.
If no `filename` is given, each file will be given a random name that doesn't
include any file extension.

**Note:** Multer will not append any file extension for you, your function
should return a filename complete with an file extension.

Each function gets passed both the request (`req`) and some information about
the file (`file`) to aid with the decision.

Note that `req.body` might not have been fully populated yet. It depends on the
order that the client transmits fields and files to the server.

#### `MemoryStorage`

The memory storage engine stores the files in memory as `Buffer` objects. It
doesn't have any options.

```javascript
var storage = multer.memoryStorage()
var upload = multer({ storage: storage })
```

When using memory storage, the file info will contain a field called
`buffer` that contains the entire file.

**WARNING**: Uploading very large files, or relatively small files in large
numbers very quickly, can cause your application to run out of memory when
memory storage is used.

### `limits`

An object specifying the size limits of the following optional properties. Multer passes this object into busboy directly, and the details of the properties can be found on [busboy's page](https://github.com/mscdex/busboy#busboy-methods).

The following integer values are available:

Key | Description | Default
--- | --- | ---
`fieldNameSize` | Max field name size | 100 bytes
`fieldSize` | Max field value size | 1MB
`fields` | Max number of non-file fields | Infinity
`fileSize` | For multipart forms, the max file size (in bytes) | Infinity
`files` | For multipart forms, the max number of file fields | Infinity
`parts` | For multipart forms, the max number of parts (fields + files) | Infinity
`headerPairs` | For multipart forms, the max number of header key=>value pairs to parse | 2000

Specifying the limits can help protect your site against denial of service (DoS) attacks.

### `fileFilter`

Set this to a function to control which files should be uploaded and which
should be skipped. The function should look like this:

```javascript
function fileFilter (req, file, cb) {

  // The function should call `cb` with a boolean
  // to indicate if the file should be accepted

  // To reject this file pass `false`, like so:
  cb(null, false)

  // To accept the file pass `true`, like so:
  cb(null, true)

  // You can always pass an error if something goes wrong:
  cb(new Error('I don\'t have a clue!'))

}
```

## Error handling

When encountering an error, multer will delegate the error to express. You can
display a nice error page using [the standard express way](http://expressjs.com/guide/error-handling.html).

If you want to catch errors specifically from multer, you can call the
middleware function by yourself.

```javascript
var upload = multer().single('avatar')

app.post('/profile', function (req, res) {
  upload(req, res, function (err) {
    if (err) {
      // An error occurred when uploading
      return
    }

    // Everything went fine
  })
})
```

# Koa-multer
# Installation

` npm install --save koa-multer `

## Usage

### **=1.x**, **100%**, working with [multer-v1.x](https://github.com/expressjs/multer) and [koa-v2.x](https://github.com/koajs/koa/tree/v2.x).

```js
const Koa = require('koa');
const route = require('koa-route');
const multer = require('koa-multer');

const app = new Koa();
const upload = multer({ dest: 'uploads/' });

app.use(route.post('/profile', upload.single('avatar')));

app.listen(3000);
```

### **=0.x**, working with `multer-v0.x`(v0.1.8 is the latset version of v0.x) and [koa-v1.x](https://github.com/koajs/koa)

```js
var koa = require('koa');
var multer = require('koa-multer');

var app = koa();

app.use(multer({ dest: './uploads/'}));

app.listen(3000);
```

## Custom storage engine

# Multer Storage Engine

Storage engines are classes that expose two functions: `_handleFile` and `_removeFile`.
Follow the template below to get started with your own custom storage engine.

When asking the user for input (such as where to save this file), always give
them the parameters `req, file, cb`, in this order. This makes it easier for
developers to switch between storage engines.

For example, in the template below, the engine saves the files to the disk. The
user tells the engine where to save the file, and this is done by
providing the `destination` parameter:

```javascript
var storage = myCustomStorage({
  destination: function (req, file, cb) {
    cb(null, '/var/www/uploads/' + file.originalname)
  }
})
```

Your engine is responsible for storing the file and returning information on how to
access the file in the future. This is done by the `_handleFile` function.

The file data will be given to you as a stream (`file.stream`). You should pipe
this data somewhere, and when you are done, call `cb` with some information on the
file.

The information you provide in the callback will be merged with multer's file object,
and then presented to the user via `req.files`.

Your engine is also responsible for removing files if an error is encountered
later on. Multer will decide which files to delete and when. Your storage class must
implement the `_removeFile` function. It will receive the same arguments as
`_handleFile`. Invoke the callback once the file has been removed.

## Template

```javascript
var fs = require('fs')

function getDestination (req, file, cb) {
  cb(null, '/dev/null')
}

function MyCustomStorage (opts) {
  opts.getDestination = (opts.destination || getDestination)
}

MyCustomStorage.prototype._handleFile = function _handleFile (req, file, cb) {
  this.getDestination(req, file, function (err, path) {
    if (err) return cb(err)

    var outStream = fs.createWriteStream(path)

    file.stream.pipe(outStream)
    outStream.on('error', cb)
    outStream.on('finish', function () {
      cb(null, {
        path: path,
        size: outStream.bytesWritten
      })
    })
  })
}

MyCustomStorage.prototype._removeFile = function _removeFile (req, file, cb) {
  fs.unlink(file.path, cb)
}

module.exports = function (opts) {
  return new MyCustomStorage(opts)
}
```


## License

[MIT](LICENSE)