aws4
-------------

[![Build Status](https://secure.travis-ci.org/mhart/aws4.png?branch=master)](http://travis-ci.org/mhart/aws4)

A small utility to sign vanilla node.js http(s) request options using Amazon's
[AWS Signature Version 4](http://docs.amazonwebservices.com/general/latest/gr/signature-version-4.html).
It also provides defaults for a number of core AWS headers and
request parameters, making it a very easy to query AWS services, or
build out a fully-featured AWS library.

Example
-------

```javascript
var http  = require('http')
  , https = require('https')
  , aws4  = require('aws4')
  , opts  = { host: 'sqs.us-east-1.amazonaws.com', path: '/?Action=ListQueues' }

// assuming AWS credentials are available from process.env
aws4.sign(opts)

// opts.headers now contains the signed AWS headers, and can be used
// in standard node.js http(s) requests

// pipe the SOAP response from SQS to stdout
http.request(opts, function(res) { res.pipe(process.stdout) }).end()

// you can pass AWS credentials in explicitly
aws4.sign(opts, { accessKeyId: '', secretAccessKey: '' })

// aws4 can infer the host from a service and region
opts = aws4.sign({ service: 'sqs', region: 'us-east-1', path: '/?Action=ListQueues' })

// aws4 can infer the HTTP method if a body is passed in
opts = aws4.sign({ service: 'iam', body: 'Action=ListGroups&Version=2010-05-08' })

// opts.method will be POST and Content-Type 'application/x-www-form-urlencoded'
https.request(opts, function(res) { res.pipe(process.stdout) }).end(opts.body)

// can specify any custom option or header as per usual
opts = aws4.sign({
  service: 'dynamodb',
  region: 'ap-southeast-2',
  method: 'POST',
  path: '/',
  headers: {
    'Content-Type': 'application/x-amz-json-1.0',
    'X-Amz-Target': 'DynamoDB_20111205.ListTables'
  },
  body: '{}'
})

// works with all other services that support Signature Version 4

opts = aws4.sign({ service: 'sts', path: '/?Action=GetSessionToken&Version=2011-06-15' })

opts = aws4.sign({
  service: 'glacier',
  path: '/-/vaults',
  headers: { 'X-Amz-Glacier-Version': '2012-06-01' }
})

opts = aws4.sign({ service: 'cloudsearch', path: '/?Action=DescribeDomains' })

```

API
---

### aws4.sign(requestOptions, [credentials])

This calculates and populates the `Authorization` header of
`requestOptions`, and any other necessary AWS headers and/or request
options. Returns `requestOptions` as a convenience for chaining.

`requestOptions` is an object holding the same options that the node.js
[http.request](http://nodejs.org/docs/latest/api/http.html#http_http_request_options_callback)
function takes.

The following properties of `requestOptions` are used in the signing or
populated if they don't already exist:

- `hostname` or `host` (will be determined from `service` and `region` if not given)
- `method` (will use `'GET'` if not given or `'POST'` if there is a `body`)
- `path` (will use `'/'` if not given)
- `body` (will use `''` if not given)
- `service` (will be calculated from `hostname` or `host` if not given)
- `region` (will be calculated from `hostname` or `host` or use `'us-east-1'` if not given)
- `headers['Host']` (will use `hostname` or `host` or be calculated if not given)
- `headers['Content-Type']` (will use `'application/x-www-form-urlencoded'` if not given and there is a `body`)
- `headers['Date']` (used to calculate the signature date if given, otherwise `new Date` is used)

Your AWS credentials (which can be found in your
[AWS console](https://portal.aws.amazon.com/gp/aws/securityCredentials))
can be specified in one of two ways:

- As the second argument, like this:

```javascript
aws4.sign(requestOptions, {
  secretAccessKey: "<your-secret-access-key>",
  accessKeyId: "<your-access-key-id>"
})
```

- From `process.env`, such as this:

```
export AWS_SECRET_ACCESS_KEY="<your-secret-access-key>"
export AWS_ACCESS_KEY_ID="<your-access-key-id>"
```

(will also use `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` if available)
