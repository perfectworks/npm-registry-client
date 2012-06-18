# npm-registry-client

The code that npm uses to talk to the registry.

It handles all the caching and HTTP calls.

## Usage

```javascript
var RegClient = require('npm-registry-client')
var client = new RegClient(options)

client.get("npm", "latest", 1000, function (er, data, raw, res) {
  // error is an error if there was a problem.
  // data is the parsed data object
  // raw is the json string
  // res is the response from couch
})
```

# Options

* `registry` **Required** {String} URL to the registry
* `cache` **Required** {String} Path to the cache folder
* `alwaysAuth` {Boolean} Auth even for GET requests.
* `auth` {String} A base64-encoded `username:password`
* `email` {String} User's email address
* `tag` {String} The default tag to use when publishing new packages.
  Default = `"latest"`
* `ca` {String} Cerficate signing authority certificates to trust.
* `strictSSL` {Boolean} Whether or not to be strict with SSL
  certificates.  Default = `true`
* `userAgent` {String} User agent header to send.  Default =
  `"node/{process.version}"`
* `log` {Object} The logger to use.  Defaults to `require("npmlog")` if
  that works, otherwise logs are disabled.

# client.request(method, where, [what], [etag], [nofollow], cb)

* `method` {String} HTTP method
* `where` {String} Path to request on the server
* `what` {Stream | Buffer | String | Object} The request body.  Objects
  that are not Buffers or Streams are encoded as JSON.
* `etag` {String} The cached ETag
* `nofollow` {Boolean} Prevent following 302/301 responses
* `cb` {Function}
  * `error` {Error | null}
  * `data` {Object} the parsed data object
  * `raw` {String} the json
  * `res` {Response Object} response from couch

Make a request to the registry.  All the other methods are wrappers
around this. one.

# client.adduser(username, password, email, cb)

* `username` {String}
* `password` {String}
* `email` {String}
* `cb` {Function}

Add a user account to the registry, or verify the credentials.

# client.get(url, [timeout], [nofollow], [staleOk], cb)

* `url` {String} The url path to fetch
* `timeout` {Number} Number of seconds old that a cached copy must be
  before a new request will be made.
* `nofollow` {Boolean} Do not follow 301/302 responses
* `staleOk` {Boolean} If there's cached data available, then return that
  to the callback quickly, and update the cache the background.

Fetches data from the registry via a GET request, saving it in
the cache folder with the ETag.

# client.publish(data, tarball, [readme], cb)

* `data` {Object} Package data
* `tarball` {String | Stream} Filename or stream of the package tarball
* `readme` {String} Contents of the README markdown file
* `cb` {Function}

Publish a package to the registry.

Note that this does not create the tarball from a folder.  However, it
can accept a gzipped tar stream or a filename to a tarball.

# client.star(package, starred, cb)

* `package` {String} Name of the package to star
* `starred` {Boolean} True to star the package, false to unstar it.
* `cb` {Function}

Star or unstar a package.

Note that the user does not have to be the package owner to star or
unstar a package, though other writes do require that the user be the
package owner.

# client.tag(project, version, tag, cb)

* `project` {String} Project name
* `version` {String} Version to tag
* `tag` {String} Tag name to apply
* `cb` {Function}

Mark a version in the `dist-tags` hash, so that `pkg@tag`
will fetch the specified version.

# client.unpublish(name, [ver], cb)

* `name` {String} package name
* `ver` {String} version to unpublish. Leave blank to unpublish all
  versions.
* `cb` {Function}

Remove a version of a package (or all versions) from the registry.  When
the last version us unpublished, the entire document is removed from the
database.

# client.upload(where, file, [etag], [nofollow], cb)

* `where` {String} URL path to upload to
* `file` {String | Stream} Either the filename or a readable stream
* `etag` {String} Cache ETag
* `nofollow` {Boolean} Do not follow 301/302 responses
* `cb` {Function}

Upload an attachment.  Mostly used by `client.publish()`.