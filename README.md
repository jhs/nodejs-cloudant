# Cloudant Node.js Client

This is the official Cloudant library for Node.js.

* [Installation and Usage](#installation-and-usage)
* [Getting Started](#getting-started)
* [API Reference](#api-reference)
* [Development](#development)
  * [Test Suite](#test-suite)
  * [Using in Other Projects](#using-in-other-projects)

## Installation and Usage

The best way to use the Cloudant client is to begin with your own Node.js project, and define this work as your dependency. In other words, put me in your package.json dependencies. The `npm` tool can do this for you, from the command line:

    $ npm install --save cloudant

Notice that your package.json will now reflect this GitHub link. Everyting is working if you can run this command with no errors:

    $ node -e 'require("cloudant"); console.log("Cloudant works");'
    Cloudant works

### Getting Started

Now it's time to begin doing real work with Cloudant and Node.js.

Initialize your Cloudant connection by supplying your *account* and *password*, and supplying a callback function to run when eveything is ready.

~~~ js
var me = 'jhs' // Set this to your own account
var password = process.env.cloudant_password

Cloudant({account:me, password:password}, function(er, cloudant) {
  if (er)
    return console.log('Error connecting to Cloudant account %s: %s', me, er.message)

  console.log('Connected to cloudant')
  cloudant.ping(function(er, reply) {
    if (er)
      return console.log('Failed to ping Cloudant. Did the network just go down?')

    console.log('Server version = %s', reply.version)
    console.log('I am %s and my roles are %j', reply.userCtx.name, reply.userCtx.roles)

    cloudant.db.list(function(er, all_dbs) {
      if (er)
        return console.log('Error listing databases: %s', er.message)

      console.log('All my databases: %s', all_dbs.join(', '))
    })
  })
})
~~~

Output:

    Connected to cloudant
    Server version = 1.0.2
    I am jhs and my roles are ["_admin","_reader","_writer"]
    All my databases: example_db, jasons_stuff, scores

Upper-case `Cloudant` is this package you load using `require()`, while lower-case `cloudant` represents an authenticated, confirmed connection to your Cloudant service.

If you omit the "password" field, you will get an "anonymous" connection: a client that sends no authentication information (no passwords, no cookies, etc.)

The `.ping()` call is for clarity. In fact, when you initialize your conneciton, you implicitly ping Cloudant, and the "pong" value is passed to you as an optional extra argument: `Cloudant({account:"A", password:"P"}, function(er, cloudant, pong_reply) { ... })`

To use this code as-is, you must first type ` export cloudant_password="<whatever>"` in your shell. This is inconvenient, and you can invent your own alternative technique;

### Security Note

**DO NOT hard-code your password and commit it to Git**. Storing your password directly in your source code (even in old, long-deleted commits) is a serious security risk to your data. Whoever gains access to your software will now also have access read, write, and delete permission to your data. Think about GitHub security bugs, or contractors, or disgruntled employees, or lost laptops at a conference. If you check in your password, all of these situations become major liabilities. (Also, note that if you follow these instructions, the `export` command with your password will likely be in your `.bash_history` now, which is kind of bad. However, if you input a space before typing the command, it will not be stored in your history.)

## TODO

  * [Authorization](#authorization)
  * [Cloudant Query](#query)
  * [Cloudant Search](#search)

## API Reference

- [Initialization](#initialization)
- [First Steps](#first-steps)
- [tutorials & screencasts](#tutorials-examples-in-the-wild--screencasts)
- [configuration](#configuration)
- [database functions](#database-functions)
	- [Cloudant.db.create(name, [callback])](#Cloudantdbcreatename-callback)
	- [Cloudant.db.get(name, [callback])](#Cloudantdbgetname-callback)
	- [Cloudant.db.destroy(name, [callback])](#Cloudantdbdestroyname-callback)
	- [Cloudant.db.list([callback])](#Cloudantdblistcallback)
	- [Cloudant.db.compact(name, [designname], [callback])](#Cloudantdbcompactname-designname-callback)
	- [Cloudant.db.replicate(source, target, [opts], [callback])](#Cloudantdbreplicatesource-target-opts-callback)
	- [Cloudant.db.changes(name, [params], [callback])](#Cloudantdbchangesname-params-callback)
	- [Cloudant.db.follow(name, [params], [callback])](#Cloudantdbfollowname-params-callback)
	- [Cloudant.use(name)](#Cloudantusename)
	- [Cloudant.request(opts, [callback])](#Cloudantrequestopts-callback)
	- [Cloudant.config](#Cloudantconfig)
	- [Cloudant.updates([params], [callback])](#Cloudantupdatesparams-callback)
	- [Cloudant.follow_updates([params], [callback])](#Cloudantfollow_updatesparams-callback)
- [document functions](#document-functions)
	- [db.insert(doc, [params], [callback])](#dbinsertdoc-params-callback)
	- [db.destroy(docname, rev, [callback])](#dbdestroydocname-rev-callback)
	- [db.get(docname, [params], [callback])](#dbgetdocname-params-callback)
	- [db.head(docname, [callback])](#dbheaddocname-callback)
	- [db.copy(src_doc, dest_doc, opts, [callback])](#dbcopysrc_doc-dest_doc-opts-callback)
	- [db.bulk(docs, [params], [callback])](#dbbulkdocs-params-callback)
	- [db.list([params], [callback])](#dblistparams-callback)
	- [db.fetch(docnames, [params], [callback])](#dbfetchdocnames-params-callback)
  - [db.fetch_revs(docnames, [params], [callback])](#dbfetch_revsdocnames-params-callback)
- [multipart functions](#multipart-functions)
	- [db.multipart.insert(doc, attachments, [params], [callback])](#dbmultipartinsertdoc-attachments-params-callback)
	- [db.multipart.get(docname, [params], [callback])](#dbmultipartgetdocname-params-callback)
- [attachments functions](#attachments-functions)
	- [db.attachment.insert(docname, attname, att, contenttype, [params], [callback])](#dbattachmentinsertdocname-attname-att-contenttype-params-callback)
	- [db.attachment.get(docname, attname, [params], [callback])](#dbattachmentgetdocname-attname-params-callback)
	- [db.attachment.destroy(docname, attname, rev, [callback])](#dbattachmentdestroydocname-attname-rev-callback)
- [views and design functions](#views-and-design-functions)
	- [db.view(designname, viewname, [params], [callback])](#dbviewdesignname-viewname-params-callback)
	- [db.show(designname, showname, doc_id, [params], [callback])](#dbshowdesignname-showname-doc_id-params-callback)
	- [db.atomic(designname, updatename, docname, [body], [callback])](#dbatomicdesignname-updatename-docname-body-callback)
	- [db.search(designname, viewname, [params], [callback])](#dbsearchdesignname-searchname-params-callback)
- [using cookie authentication](#using-cookie-authentication)
- [advanced features](#advanced-features)
	- [extending Cloudant](#extending-Cloudant)
	- [pipes](#pipes)
- [tests](#tests)

## First Steps

To use Cloudant, require this package in your code, and run the function with your account name and password. (And see the [security note](#security-note) about placing your password into your source code.

~~~ js
var Cloudant = require('cloudant')

Cloudant({account:me, password:password}, function(er, cloudant) {
  if (er)
    return console.log('Error connecting to Cloudant account %s: %s', me, er.message)

  console.log('Connected to cloudant')
~~~

to create a new database:

~~~ js
Cloudant.db.create('alice');
~~~

and to use it:

~~~ js
var alice = Cloudant.db.use('alice');
~~~

in this examples we didn't specify a `callback` function, the absence of a
callback means _"do this, ignore what happens"_.
in `Cloudant` the callback function receives always three arguments:

* `err` - the error, if any
* `body` - the http _response body_ from couchdb, if no error.
  json parsed body, binary for non json responses
* `header` - the http _response header_ from couchdb, if no error


a simple but complete example using callbacks is:

~~~ js
var Cloudant = require('Cloudant')('http://localhost:5984');

// clean up the database we created previously
Cloudant.db.destroy('alice', function() {
  // create a new database
  Cloudant.db.create('alice', function() {
    // specify the database we are going to use
    var alice = Cloudant.use('alice');
    // and insert a document in it
    alice.insert({ crazy: true }, 'rabbit', function(err, body, header) {
      if (err) {
        console.log('[alice.insert] ', err.message);
        return;
      }
      console.log('you have inserted the rabbit.')
      console.log(body);
    });
  });
});
~~~

if you run this example(after starting couchdb) you will see:

    you have inserted the rabbit.
    { ok: true,
      id: 'rabbit',
      rev: '1-6e4cb465d49c0368ac3946506d26335d' }

you can also see your document in [futon](http://localhost:5984/_utils).

## configuration

configuring Cloudant to use your database server is as simple as:

~~~ js
var Cloudant   = require('Cloudant')('http://localhost:5984')
  , db     = Cloudant.use('foo')
  ;
~~~

however if you don't need to instrument database objects you can simply:

~~~ js
// Cloudant parses the url and knows this is a database
var db = require('Cloudant')('http://localhost:5984/foo');
~~~

you can also pass options to the require:

~~~ js
// Cloudant parses the url and knows this is a database
var db = require('Cloudant')('http://localhost:5984/foo');
~~~

to specify further configuration options you can pass an object literal instead:

~~~ js
// Cloudant parses the url and knows this is a database
var db = require('Cloudant')(
  { "url"             : "http://localhost:5984/foo"
  , "request_defaults" : { "proxy" : "http://someproxy" }
  , "log"             : function (id, args) {
      console.log(id, args);
    }
  });
~~~
Please check [request] for more information on the defaults. They support features like cookie jar, proxies, ssl, etc.

### pool size and open sockets

a very important configuration parameter if you have a high traffic website and are using Cloudant is setting up the `pool.size`. by default, the node.js http global agent (client) has a certain size of active connections that can run simultaneously, while others are kept in a queue. pooling can be disabled by setting the `agent` property in `request_defaults` to false, or adjust the global pool size using:

~~~ js
http.globalAgent.maxSockets = 20;
~~~

you can also increase the size in your calling context using `request_defaults` if this is problematic. refer to the [request] documentation and examples for further clarification.

here's an example explicitly using the keep alive agent (installed using `npm install agentkeepalive`), especially useful to limit your open sockets when doing high-volume access to couchdb on localhost:

~~~ js
var agentkeepalive = require('agentkeepalive');
var myagent = new agentkeepalive({
    maxSockets: 50
  , maxKeepAliveRequests: 0
  , maxKeepAliveTime: 30000
  });

var db = require('Cloudant')(
  { "url"              : "http://localhost:5984/foo"
  , "request_defaults" : { "agent" : myagent }
  });
~~~


## database functions

### Cloudant.db.create(name, [callback])

creates a couchdb database with the given `name`.

~~~ js
Cloudant.db.create('alice', function(err, body) {
  if (!err) {
    console.log('database alice created!');
  }
});
~~~

### Cloudant.db.get(name, [callback])

get informations about `name`.

~~~ js
Cloudant.db.get('alice', function(err, body) {
  if (!err) {
    console.log(body);
  }
});
~~~

### Cloudant.db.destroy(name, [callback])

destroys `name`.

~~~ js
Cloudant.db.destroy('alice');
~~~

even though this examples looks sync it is an async function.

### Cloudant.db.list([callback])

lists all the databases in couchdb

~~~ js
Cloudant.db.list(function(err, body) {
  // body is an array
  body.forEach(function(db) {
    console.log(db);
  });
});
~~~

### Cloudant.db.compact(name, [designname], [callback])

compacts `name`, if `designname` is specified also compacts its
views.

### Cloudant.db.replicate(source, target, [opts], [callback])

replicates `source` on `target` with options `opts`. `target`
has to exist, add `create_target:true` to `opts` to create it prior to
replication.

~~~ js
Cloudant.db.replicate('alice', 'http://admin:password@otherhost.com:5984/alice',
                  { create_target:true }, function(err, body) {
    if (!err)
      console.log(body);
});
~~~

### Cloudant.db.changes(name, [params], [callback])

asks for the changes feed of `name`, `params` contains additions
to the query string.

~~~ js
Cloudant.db.changes('alice', function(err, body) {
  if (!err) {
    console.log(body);
  }
});
~~~

### Cloudant.db.follow(name, [params], [callback])

uses [follow] to create a solid changes feed. please consult follow documentation for more information as this is a very complete api on it's own

~~~ js
var feed = db.follow({since: "now"});
feed.on('change', function (change) {
  console.log("change: ", change);
});
feed.follow();
process.nextTick(function () {
  db.insert({"bar": "baz"}, "bar");
});
~~~

### Cloudant.use(name)

creates a scope where you operate inside `name`.

~~~ js
var alice = Cloudant.use('alice');
alice.insert({ crazy: true }, 'rabbit', function(err, body) {
  // do something
});
~~~

### Cloudant.db.use(name)

alias for `Cloudant.use`

### Cloudant.db.scope(name)

alias for `Cloudant.use`

### Cloudant.scope(name)

alias for `Cloudant.use`

### Cloudant.request(opts, [callback])

makes a request to couchdb, the available `opts` are:

* `opts.db` – the database name
* `opts.method` – the http method, defaults to `get`
* `opts.path` – the full path of the request, overrides `opts.doc` and
  `opts.att`
* `opts.doc` – the document name
* `opts.att` – the attachment name
* `opts.params` – query string parameters, appended after any existing `opts.path`, `opts.doc`, or `opts.att`
* `opts.content_type` – the content type of the request, default to `json`
* `opts.headers` – additional http headers, overrides existing ones
* `opts.body` – the document or attachment body
* `opts.encoding` – the encoding for attachments
* `opts.multipart` – array of objects for multipart request

### Cloudant.relax(opts, [callback])

alias for `Cloudant.request`

### Cloudant.dinosaur(opts, [callback])

alias for `Cloudant.request`

                    _
                  / '_)  WAT U SAY!
         _.----._/  /
        /          /
      _/  (   | ( |
     /__.-|_|--|_l

### Cloudant.config

an object containing the Cloudant configurations, possible keys are:

* `url` - the couchdb url
* `db` - the database name


### Cloudant.updates([params], [callback])

listen to db updates, the available `params` are:
  
* `params.feed` – Type of feed. Can be one of
 * `longpoll`: Closes the connection after the first event.
 * `continuous`: Send a line of JSON per event. Keeps the socket open until timeout.
 * `eventsource`: Like, continuous, but sends the events in EventSource format.
* `params.timeout` – Number of seconds until CouchDB closes the connection. Default is 60.
* `params.heartbeat` – Whether CouchDB will send a newline character (\n) on timeout. Default is true.


### Cloudant.follow_updates([params], [callback])

uses [follow](https://github.com/iriscouch/follow) to create a solid
[`_db_updates`](http://docs.couchdb.org/en/latest/api/server/common.html?highlight=db_updates#get--_db_updates) feed.
please consult follow documentation for more information as this is a very complete api on it's own

~~~js
var feed = Cloudant.follow_updates({since: "now"});
feed.on('change', function (change) {
  console.log("change: ", change);
});
feed.follow();
process.nextTick(function () {
  Cloudant.db.create('alice');
});
~~~

## document functions

### db.insert(doc, [params], [callback])

inserts `doc` in the database with  optional `params`. if params is a string, its assumed as the intended document name. if params is an object, its passed as query string parameters and `doc_name` is checked for defining the document name.

~~~ js
var alice = Cloudant.use('alice');
alice.insert({ crazy: true }, 'rabbit', function(err, body) {
  if (!err)
    console.log(body);
});
~~~

### db.destroy(docname, rev, [callback])

removes revision `rev` of `docname` from couchdb.

~~~ js
alice.destroy('rabbit', '3-66c01cdf99e84c83a9b3fe65b88db8c0', function(err, body) {
  if (!err)
    console.log(body);
});
~~~

### db.get(docname, [params], [callback])

gets `docname` from the database with optional query string
additions `params`.

~~~ js
alice.get('rabbit', { revs_info: true }, function(err, body) {
  if (!err)
    console.log(body);
});
~~~

### db.head(docname, [callback])

same as `get` but lightweight version that returns headers only.

~~~ js
alice.head('rabbit', function(err, _, headers) {
  if (!err)
    console.log(headers);
});
~~~

### db.copy(src_doc, dest_doc, opts, [callback])

`copy` the contents (and attachments) of a document
to a new document, or overwrite an existing target document

~~~ js
alice.copy('rabbit', 'rabbit2', { overwrite: true }, function(err, _, headers) {
  if (!err)
    console.log(headers);
});
~~~


### db.bulk(docs, [params], [callback])

bulk operations(update/delete/insert) on the database, refer to the
[couchdb doc](http://wiki.apache.org/couchdb/HTTP_Bulk_Document_API).

### db.list([params], [callback])

list all the docs in the database with optional query string additions `params`.

~~~ js
alice.list(function(err, body) {
  if (!err) {
    body.rows.forEach(function(doc) {
      console.log(doc);
    });
  }
});
~~~

### db.fetch(docnames, [params], [callback])

bulk fetch of the database documents, `docnames` are specified as per
[couchdb doc](http://wiki.apache.org/couchdb/HTTP_Bulk_Document_API).
additional query string `params` can be specified, `include_docs` is always set
to `true`.

### db.fetch_revs(docnames, [params], [callback])

bulk fetch of the revisions of the database documents, `docnames` are specified as per
[couchdb doc](http://wiki.apache.org/couchdb/HTTP_Bulk_Document_API).
additional query string `params` can be specified, this is the same method as fetch but
 `include_docs` is not automatically set to `true`.

## multipart functions

### db.multipart.insert(doc, attachments, [params], [callback])

inserts a `doc` together with `attachments` and optional `params`. if params is a string, its assumed as the intended document name. if params is an object, its passed as query string parameters and `doc_name` is checked for defining the document name.
 refer to the [doc](http://wiki.apache.org/couchdb/HTTP_Document_API#Multiple_Attachments) for more details.
 `attachments` must be an array of objects with `name`, `data` and `content_type` properties.

~~~ js
var fs = require('fs');

fs.readFile('rabbit.png', function(err, data) {
  if (!err) {
    alice.multipart.insert({ foo: 'bar' }, [{name: 'rabbit.png', data: data, content_type: 'image/png'}], 'mydoc', function(err, body) {
        if (!err)
          console.log(body);
    });
  }
});
~~~

### db.multipart.get(docname, [params], [callback])

get `docname` together with its attachments via `multipart/related` request with optional query string additions
`params`. refer to the
 [doc](http://wiki.apache.org/couchdb/HTTP_Document_API#Getting_Attachments_With_a_Document) for more details.
 the multipart response body is a `Buffer`.

~~~ js
alice.multipart.get('rabbit', function(err, buffer) {
  if (!err)
    console.log(buffer.toString());
});
~~~

## attachments functions

### db.attachment.insert(docname, attname, att, contenttype, [params], [callback])

inserts an attachment `attname` to `docname`, in most cases
 `params.rev` is required. refer to the
 [doc](http://wiki.apache.org/couchdb/HTTP_Document_API) for more details.

~~~ js
var fs = require('fs');

fs.readFile('rabbit.png', function(err, data) {
  if (!err) {
    alice.attachment.insert('rabbit', 'rabbit.png', data, 'image/png',
      { rev: '12-150985a725ec88be471921a54ce91452' }, function(err, body) {
        if (!err)
          console.log(body);
    });
  }
});
~~~

or using `pipe`:

~~~ js
var fs = require('fs');

fs.createReadStream('rabbit.png').pipe(
    alice.attachment.insert('new', 'rab.png', null, 'image/png')
);
~~~

### db.attachment.get(docname, attname, [params], [callback])

get `docname`'s attachment `attname` with optional query string additions
`params`.

~~~ js
var fs = require('fs');

alice.attachment.get('rabbit', 'rabbit.png', function(err, body) {
  if (!err) {
    fs.writeFile('rabbit.png', body);
  }
});
~~~

or using `pipe`:

~~~ js
var fs = require('fs');

alice.attachment.get('rabbit', 'rabbit.png').pipe(fs.createWriteStream('rabbit.png'));
~~~

### db.attachment.destroy(docname, attname, rev, [callback])

destroy attachment `attname` of `docname`'s revision `rev`.

~~~ js
alice.attachment.destroy('rabbit', 'rabbit.png',
    '1-4701d73a08ce5c2f2983bf7c9ffd3320', function(err, body) {
      if (!err)
        console.log(body);
});
~~~

## views and design functions

### db.view(designname, viewname, [params], [callback])

calls a view of the specified design with optional query string additions
`params`. if you're looking to filter the view results by key(s) pass an array of keys, e.g
`{ keys: ['key1', 'key2', 'key_n'] }`, as `params`.

~~~ js
alice.view('characters', 'crazy_ones', function(err, body) {
  if (!err) {
    body.rows.forEach(function(doc) {
      console.log(doc.value);
    });
  }
});
~~~

### db.view_with_list(designname, viewname, listname, [params], [callback])

calls a list function feeded by the given view of the specified design document.

~~~ js
alice.view_with_list('characters', 'crazy_ones', 'my_list', function(err, body) {
  if (!err) {
    console.log(body);
  }
});
~~~

### db.show(designname, showname, doc_id, [params], [callback])

calls a show function of the specified design for the document specified by doc_id with
optional query string additions `params`.

~~~ js
alice.show('characters', 'format_doc', '3621898430', function(err, doc) {
  if (!err) {
    console.log(doc);
  }
});
~~~
take a look at the [couchdb wiki](http://wiki.apache.org/couchdb/Formatting_with_Show_and_List#Showing_Documents)
for possible query paramaters and more information on show functions.

### db.atomic(designname, updatename, docname, [body], [callback])

calls the design's update function with the specified doc in input.

~~~ js
db.atomic("update", "inplace", "foobar",
{field: "foo", value: "bar"}, function (error, response) {
  assert.equal(error, undefined, "failed to update");
  assert.equal(response.foo, "bar", "update worked");
});
~~~

Note that the data is sent in the body of the request.
An example update handler follows:

~~~ js
"updates": {
  "in-place" : "function(doc, req) {
      var field = req.form.field;
      var value = req.form.value;
      var message = 'set '+field+' to '+value;
      doc[field] = value;
      return [doc, message];
  }"
~~~

### db.search(designname, searchname, [params], [callback])

calls a view of the specified design with optional query string additions `params`.

~~~ js
alice.search('characters', 'crazy_ones', { q: 'cat' }, function(err, doc) {
  if (!err) {
    console.log(doc);
  }
});
~~~

check out the tests for a fully functioning example.

## using cookie authentication

Cloudant supports making requests using couchdb's [cookie authentication](http://guide.couchdb.org/editions/1/en/security.html#cookies) functionality. there's a [step-by-step guide here](http://codetwizzle.com/articles/couchdb-cookie-authentication-nodejs-Cloudant/), but essentially you just:

~~~ js
var Cloudant     = require('Cloudant')('http://localhost:5984')
  , username = 'user'
  , userpass = 'pass'
  , callback = console.log // this would normally be some callback
  , cookies  = {} // store cookies, normally redis or something
  ;

Cloudant.auth(username, userpass, function (err, body, headers) {
  if (err) {
    return callback(err);
  }

  if (headers && headers['set-cookie']) {
    cookies[user] = headers['set-cookie'];
  }

  callback(null, "it worked");
});
~~~

reusing a cookie:

~~~ js
var auth = "some stored cookie"
  , callback = console.log // this would normally be some callback
  , alice = require('Cloudant')(
    { url : 'http://localhost:5984/alice', cookie: 'AuthSession=' + auth });
  ;

alice.insert(doc, function (err, body, headers) {
  if (err) {
    return callback(err);
  }

  // change the cookie if couchdb tells us to
  if (headers && headers['set-cookie']) {
    auth = headers['set-cookie'];
  }

  callback(null, "it worked");
});
~~~

getting current session:

~~~javascript
var Cloudant = require('Cloudant')({url: 'http://localhost:5984', cookie: 'AuthSession=' + auth});

Cloudant.session(function(err, session) {
  if (err) {
    return console.log('oh noes!')
  }

  console.log('user is %s and has these roles: %j',
    session.userCtx.user, session.userCtx.roles);
});
~~~


## advanced features

### extending Cloudant

Cloudant is minimalistic but you can add your own features with
`Cloudant.request(opts, callback)`

for example, to create a function to retrieve a specific revision of the
`rabbit` document:

~~~ js
function getrabbitrev(rev, callback) {
  Cloudant.request({ db: 'alice',
                 doc: 'rabbit',
                 method: 'get',
                 params: { rev: rev }
               }, callback);
}

getrabbitrev('4-2e6cdc4c7e26b745c2881a24e0eeece2', function(err, body) {
  if (!err) {
    console.log(body);
  }
});
~~~
### pipes

you can pipe in Cloudant like in any other stream.
for example if our `rabbit` document has an attachment with name `picture.png`
(with a picture of our white rabbit, of course!) you can pipe it to a `writable
stream`

~~~ js
var fs = require('fs'),
    Cloudant = require('Cloudant')('http://127.0.0.1:5984/');
var alice = Cloudant.use('alice');
alice.attachment.get('rabbit', 'picture.png').pipe(fs.createWriteStream('/tmp/rabbit.png'));
~~~

then open `/tmp/rabbit.png` and you will see the rabbit picture.


## tutorials, examples in the wild & screencasts

* article: [Cloudant - a minimalistic couchdb client for nodejs](http://writings.nunojob.com/2011/08/Cloudant-minimalistic-couchdb-client-for-nodejs.html)
* article: [getting started with node.js and couchdb](http://writings.nunojob.com/2011/09/getting-started-with-nodejs-and-couchdb.html)
* article: [document update handler support](http://jackhq.tumblr.com/post/16035106690/Cloudant-v1-2-x-document-update-handler-support-v1-2-x)
* article: [Cloudant 3](http://writings.nunojob.com/2012/05/Nano-3.html)
* article: [securing a site with couchdb cookie authentication using node.js and Cloudant](http://codetwizzle.com/articles/couchdb-cookie-authentication-nodejs-Cloudant/)
* article: [adding copy to Cloudant](http://blog.jlank.com/2012/07/04/adding-copy-to-Cloudant/)
* article: [how to update a document with Cloudant](http://writings.nunojob.com/2012/07/How-To-Update-A-Document-With-Nano-The-CouchDB-Client-for-Node.js.html)
* article: [thoughts on development using couchdb with node.js](http://tbranyen.com/post/thoughts-on-development-using-couchdb-with-nodejs)
* example in the wild: [Cloudantblog](https://github.com/grabbeh/Cloudantblog)

## roadmap

check [issues][2]

## tests

to run (and configure) the test suite simply:

~~~ sh
cd Cloudant
npm install
npm test
~~~

after adding a new test you can run it individually (with verbose output) using:

~~~ sh
Cloudant_env=testing node tests/doc/list.js list_doc_params
~~~

where `list_doc_params` is the test name.

## meta

                    _
                  / _) roar! i'm a vegan!
           .-^^^-/ /
        __/       /
       /__.|_|-|_|     cannes est superb

* code: `git clone git://github.com/dscape/Cloudant.git`
* home: <http://github.com/dscape/Cloudant>
* bugs: <http://github.com/dscape/Cloudant/issues>
* build: [![build status](https://secure.travis-ci.org/dscape/Cloudant.png)](http://travis-ci.org/dscape/Cloudant)
* deps: [![deps status](https://david-dm.org/dscape/Cloudant.png)](https://david-dm.org/dscape/Cloudant)
* chat: <https://gitter.im/dscape/Cloudant>

`(oo)--',-` in [caos][3]

[1]: http://npmjs.org
[2]: http://github.com/dscape/Cloudant/issues
[3]: http://caos.di.uminho.pt/
[4]: https://github.com/dscape/Cloudant/blob/master/cfg/couch.example.js
[follow]: https://github.com/iriscouch/follow
[request]:  https://github.com/mikeal/request

## license

copyright 2011 nuno job <nunojob.com> (oo)--',--

licensed under the apache license, version 2.0 (the "license");
you may not use this file except in compliance with the license.
you may obtain a copy of the license at

    http://www.apache.org/licenses/LICENSE-2.0.html

unless required by applicable law or agreed to in writing, software
distributed under the license is distributed on an "as is" basis,
without warranties or conditions of any kind, either express or implied.
see the license for the specific language governing permissions and
limitations under the license.

## Development

To join the effort developing this project, start from our GitHub page: https://github.com/cloudant/nodejs-cloudant

First clone this project from GitHub, and then install its dependencies using npm.

    $ git clone https://github.com/cloudant/nodejs-cloudant
    $ npm install

## Test Suite

We use npm to handle running the test suite. To run the comprehensive test suite, just run `npm test`. However, to run only the Cloudant-specific bits, we have a custom `test-cloudant` script.

    $ npm run test-cloudant

    > cloudant@5.10.1 test-cloudant /Users/jhs/src/cloudant/nodejs-cloudant
    > env NOCK=on sh tests/cloudant/run-tests.sh

    Test against mocked local database

      /tests/cloudant/auth.js

    ✔ 5/5 cloudant:generate_api_key took 196ms
    ✔ 3/3 cloudant:set_permissions took 7ms
    ✔ 8/8 summary took 224ms
    <...cut a bunch of test output...>

This runs against a local "mock" web server, called Nock. However the test suite can also run against a live Cloudant service. I have registered "nodejs.cloudant.com" for this purpose. To use it, run the `test-cloudant-live` script.

    $ npm run test-cloudant-live

    > cloudant@5.10.1 test-cloudant-live /Users/jhs/src/cloudant/nodejs-cloudant
    > sh tests/cloudant/run-tests.sh

    Test against mocked local database

      /tests/cloudant/auth.js

    ✔ 5/5 cloudant:generate_api_key took 192ms
    ✔ 3/3 cloudant:set_permissions took 7ms
    ✔ 8/8 summary took 221ms
    <...cut a bunch of test output...>

Unfortunately you need to know the password.

    $ npm run test-cloudant-live

    > cloudant@5.10.1 test-cloudant-live /Users/jhs/src/cloudant/nodejs-cloudant
    > sh tests/cloudant/run-tests.sh

    Test against remote Cloudant database
    No password configured for remote Cloudant database. Please run:

    npm config set cloudant_password "<your-password>"

    npm ERR! cloudant@5.10.1 test-cloudant-live: `sh tests/cloudant/run-tests.sh`
    <...cut npm error messages...>

Get the password from Jason somehow, and set it as an npm variable.

    # Note the leading space to keep this command out of the Bash history.
    $  npm config set cloudant_password "ask jason for the password" # <- Not the real password
    $ npm run test-cloudant-live
    <...cut successful test suite run...>

## Using in Other Projects

If you work on this project plus another one, your best bet is to clone from GitHub and then *link* this project to your other one. With linking, your other project depends on this one; but instead of a proper install, npm basically symlinks this project into the right place.

Go to this project and "link" it into the global namespace (sort of an "export").

    $ cd cloudant
    $ npm link
    /Users/jhs/.nvm/v0.10.25/lib/node_modules/cloudant -> /Users/jhs/src/cloudant/nodejs-cloudant

Go to your project and "link" it into there (sort of an "import").

    $ cd ../my-project
    $ npm link cloudant
    /Users/jhs/src/my-project/node_modules/cloudant -> /Users/jhs/.nvm/v0.10.25/lib/node_modules/cloudant -> /Users/jhs/src/cloudant/nodejs-cloudant

Now your project has the dependency in place, however you can work on both of them in tandem.

[nano]: https://github.com/dscape/nano
[query]: http://docs.cloudant.com/api/cloudant-query.html
[search]: http://docs.cloudant.com/api/search.html
[auth]: http://docs.cloudant.com/api/authz.html
[issues]: https://github.com/cloudant/nodejs-cloudant/issues
