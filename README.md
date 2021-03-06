# envigor

envigor is a module that creates a configuration object based on what
environment variables are available. This is useful for consolidating possible
configuration options for apps that run in

(It was originally 'enviguration', which made equally no sense and was
consequently thrown out for being needlessly long.)

## Contributing

Feel free to submit patches to add any service / interface you know well enough
to configure. Before doing so, try to get a feel for how things tend to be
laid out by looking over the rest of the documentation and code.

## Usage

Say you're making an Express app with MongoDB, that you're writing for a
service like Heroku, with addons like MongoLab to provide your more general
services like MongoDB. By using envigor, you can set your app up to find a
configured MongoDB service with complete service agnosticism:

```bash
npm install --save envigor mongodb
heroku addons:add mongolab
```

server.js:
```js
var http = require('http')
var cfg = require('envigor')();

var app = require('./app.js')(cfg);
http.createServer(app).listen(cfg.port || 5000, function() {
  console.log("Listening on " + port);
});
```

app.js:
```js
var express = require('express');
module.exports = function(cfg){
  var app = express();

  mongodb.MongoClient.connect(cfg.mongodb.url,function(err,db){
    if(err) throw err; else app.set('db',db)
  }

  // ...your routes here...

  return app;
}
```


## Names

The conventional names for the configuration variables queried for are based on
the variables and names used by services' Heroku addons, as well as example
variable names commonly used in setup documentation and tutorials.

Following this logic, envigor doesn't search for every possible permutation of
synonyms and underscores - only the ones it's reasonably likely to encounter
based on precedent. If a variable you set isn't getting picked up, check that
it's set at the *exact* name where envigor is looking for it, underscores and
all. (Specifically, note that *different services place underscores in
different places for the same phrases* , such as "APIKEY" and "API_KEY".)

Object names are primarily set based on most common phrasing across all
instances (such as "username" over "user" or "login"), followed by any
additional names used by other popular consumers (such as "user" for SMTP to
match the configuration names used by [Nodemailer][]). Specific services will
also often have other names set when a common variable name doesn't match the
primary name used by envigor.

[Nodemailer]: https://github.com/andris9/Nodemailer

In lieu of any interface-specific confuration options (eg. SMTP_PASSWORD),
services that provide general interfaces (eg. SMTP or MongoDB) are used to set
that interface's configuration options. In the (uncommon) event that more than
one service that provides a specific interface has a defined configuration,
precedence is defined in an arbitrary order roughly based on who I like best
(the order used in this documentation).

Default values are provided when part of a service is configured but not the
others, like an SMTP service without a hostname. However, objects and defaults
are not constructed if no service is specified.

## Generalized configuration options

### port

The value of the almost-inescapable `PORT` variable, used for setting the port
for a server to listen on.

### smtp

**Provided by:** mandrill, postmark, sendgrid, mailgun

- **username:** `SMTP_USERNAME` || mandrill.username || postmark.apiKey
  || sendgrid.username || mailgun.smtp.username
- **user, auth.user:** Same as **username** (to match [Nodemailer][]).
- **password:** `SMTP_PASSWORD` || mandrill.apiKey || postmark.apiKey
  || sendgrid.password || mailgun.smtp.password
- **pass, auth.pass:** Same as **password** (to match [Nodemailer][]).
- **hostname:** `SMTP_HOSTNAME` || `SMTP_HOST` || `SMTP_SERVER`
  || 'smtp.mandrillapp.com' (mandrill) || 'smtp.postmarkapp.com' (postmark)
  || 'smtp.sendgrid.net' (sendgrid)
  || (mailgun.smtp.hostname || 'smtp.mailgun.org') (mailgun)
- **host:** Same as **hostname** (to match [Nodemailer][]).
- **server:** Same as **hostname** (to match `SMTP_SERVER`).
- **port:** `SMTP_PORT` || '587' (mandrill, sendgrid) || '2525' (postmark)
  || (mailgun.smtp.port || '587') (mailgun)
- **service:** `SMTP_SERVICE` || 'mandrill', 'postmark', 'sendgrid', or
  'mailgun', depending on which service (if any) is being used.

### mongodb

**Provided by:** mongolab, mongohq

- **url:** `MONGODB_URL` || mongolab.url || mongohq.url
- **service:** `MONGODB_SERVICE` || 'mongolab' or 'mongohq', depending on which
  service (if any) is being used.

### redis

**Provided by:** rediscloud, redistogo, myredis, openredis, redisgreen

- **url:** `REDIS_URL` || rediscloud.url || redistogo.url || myredis.url
  || openredis.url || redisgreen.url || constructed, see next section
- **service:** `REDIS_SERVICE` || 'rediscloud', 'redistogo', 'myredis',
  'openredis', or 'redisgreen', depending on which service (if any) is being
  used.

Redis uses URLs of the convention `redis://database:password@hostname:port`.
If **url** is determined from the sources described above, envigor will parse
these components from that URL. Otherwise, the following parameters will be
taken from the specified environment variables and the **url** will be
constructed from their values.

- **hostname:** `REDIS_HOSTNAME` || `REDIS_HOST` || `REDIS_SERVER`
  || 'localhost'
- **port:** `REDIS_PORT` || '6379'
- **password:** `REDIS_PASSWORD`
- **database:** `REDIS_DATABASE`

## Service-specific options

### aws

Amazon Web Services

- **accessKey**: `AWS_ACCESS_KEY_ID` || `AWS_ACCESS_KEY`
- **secret**: `AWS_SECRET` || `AWS_SECRET_KEY` || `AWS_SECRET_ACCESS_KEY`

### s3

Amazon's ubiquitous S3 service.

- **accessKey**: `S3_ACCESS_KEY` || `S3_KEY` || aws.accessKey
- **key:** Same as **accessKey** (to match [knox][]).
- **secret**: `S3_SECRET` || `S3_SECRET_KEY` || aws.secret
- **bucket**: `S3_BUCKET` || `S3_BUCKET_NAME`
- **endpoint**: `S3_ENDPOINT` || 's3.amazonaws.com'

[knox]: https://github.com/LearnBoost/knox

### facebook

Facebook App configuration

- **appId**: `FACEBOOK_APP_ID`
- **secret**: `FACEBOOK_SECRET` || `FACEBOOK_APP_SECRET`
- **clientID**: Same as **appId** (to match [passport-facebook][]).
- **clientSecret**: Same as **secret** (to match [passport-facebook][]).

[passport-facebook]: https://github.com/jaredhanson/passport-facebook

### twitter

Twitter application OAuth credentials

- **key**: `TWITTER_CONSUMER_KEY` || `TWITTER_KEY`
- **secret**: `TWITTER_CONSUMER_SECRET` || `TWITTER_SECRET`
- **consumerKey**: Same as **key** (to match [passport-twitter][]).
- **consumerSecret**: Same as **secret** (to match [passport-twitter][]).

[passport-twitter]: https://github.com/jaredhanson/passport-twitter

### twilio

The [Twilio](http://www.twilio.com) cloud communications platform

- **accountSid**: `TWILIO_ACCOUNT_SID`
- **authToken**: `TWILIO_AUTH_TOKEN`
- **number**: `TWILIO_NUMBER`

### mandrill

[Mandrill](http://mandrill.com/) by MailChimp

**Provides:** smtp

- **username:** `MANDRILL_USERNAME`
- **password:** `MANDRILL_APIKEY`

### postmark

https://postmarkapp.com

**Provides:** smtp

- **apiKey:** `POSTMARK_API_KEY`
- **inboundAddress:** `POSTMARK_INBOUND_ADDRESS`

### sendgrid

SendGrid (http://sendgrid.com)

**Provides:** smtp

- **username:** `SENDGRID_USERNAME`
- **password:** `SENDGRID_PASSWORD`

### mailgun

[Mailgun](http://www.mailgun.com/): Programmable Mail Servers

**Provides:** smtp

- **apiKey:** `MAILGUN_API_KEY`
- **smtp**:
    - **username:** `MAILGUN_SMTP_LOGIN`
    - **user:** Same as **username** (to match [Nodemailer][]).
    - **login:** Same as **username** (to match `MAILGUN_SMTP_LOGIN`).
    - **password:** `MAILGUN_SMTP_PASSWORD`
    - **pass:** Same as **password** (to match [Nodemailer][]).
    - **hostname:** `MAILGUN_SMTP_SERVER`
    - **host:** Same as **hostname** (to match [Nodemailer][]).
    - **server:** Same as **hostname** (to match `MAILGUN_SMTP_SERVER`).
    - **port:** `MAILGUN_SMTP_PORT`

### mongolab

MongoLab MongoDB-as-a-Service - https://mongolab.com/

**Provides:** mongodb

- **url:** `MONGOLAB_URI`
- **uri:** Same as **url** (to match `MONGOLAB_URI`).

### mongohq

http://mongohq.com

**Provides:** mongodb

- **url:** `MONGOHQ_URL`

### rediscloud

Redis Cloud powered by Garantia Data - http://redis-cloud.com/

**Provides:** redis

- **url:** `REDISCLOUD_URL`
- **hostname, password, port, database:** Parsed from **url**.
  See the section on 'redis'.

### redistogo

[Redis To Go](http://redistogo.com/) Simple Redis Hosting

**Provides:** redis

- **url:** `REDISTOGO_URL`
- **hostname, password, port, database:** Parsed from **url**.
  See the section on 'redis'.

### myredis

[MyRedis](http://myredis.com/)

**Provides:** redis

- **url:** `MYREDIS_URL`
- **hostname, password, port, database:** Parsed from **url**.
  See the section on 'redis'.

### openredis

https://openredis.com/

**Provides:** redis

- **url:** `OPENREDIS_URL`
- **hostname, password, port, database:** Parsed from **url**.
  See the section on 'redis'.

### redisgreen

Dedicated Redis hosting - http://www.redisgreen.net/

**Provides:** redis

- **url:** `REDISGREEN_URL`
- **hostname, password, port, database:** Parsed from **url**.
  See the section on 'redis'.
