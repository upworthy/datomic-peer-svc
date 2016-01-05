## Datomic Peer Buildpack

**Datomic Peer Buildpack** installs [Datomic REST service][0] bound to
`localhost:${DATOMIC_REST_PORT:-8777}` running as a daemon on the same
dyno as the app. This is useful for non-JVM Heroku apps that wish to
use Datomic.

Naturally, the app and the datomic peer (REST service) will compete
for resources of a dyno. The buildpack addresses that by restricting
memory resources dedicated to Datomic REST service based on Heroku
dyno size.

Add this buildpack to your APP like so:

```
heroku buildpacks:add https://github.com/upworthy/datomic-peer-svc.git -a APP
```

Ensure your app has Datomic-related configuration, mutatis mutandis:

```
heroku config:set -a APP \
  DATOMIC_TECH_CONTACT=you@example.com \
  DATOMIC_DOWNLOAD_KEY=secret \
  DATOMIC_VERSION=0.9.5302 \
  DATOMIC_STORAGE_SPECS="alias1 datomic:ddb://<YOUR-AWS-REGION>/<YOUR-STORAGE-TABLE>/"
```

You may specify more than one Datomic storage using the command line
syntax of Datomic's `bin/console` program: `DATOMIC_STORAGE_SPECS="alias1 uri1 alias2 uri2 ..."`

For example: `DATOMIC_STORAGE_SPECS="db1 datomic:ddb://us-east-1/abc db2 datomic:ddb://us-west-2/xyz"`

[0]: http://docs.datomic.com/rest.html

## Datomic Peer Service

**Datomic Peer Service** allows you to run Datomic Console or REST API
as a Heroku app. It consists of two components:

 - [oauth2_proxy,][1] a proxy that ensures only authorized
   Google-authenticated users may access your Datomic infrastructure
   (aka **OA2P**).
 - [Datomic Console,][2] a graphical UI that makes it easy to work
   with Datomic databases. It provides tools for exploring schema,
   building and executing queries, examining transactions and more.

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/upworthy/datomic-peer-svc)

[1]: https://github.com/bitly/oauth2_proxy
[2]: http://docs.datomic.com/console.html

## Dynamo DB Storage

You must specify `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
credentials of an AWS IAM user with the following statement among its
policies:

```json
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem",
                "dynamodb:BatchGetItem",
                "dynamodb:Scan",
                "dynamodb:Query"
            ],
            "Resource": "arn:aws:dynamodb:*:<YOUR-AWS-ACCOUNT>:table/<YOUR-STORAGE-TABLE>"
        }
    ]
}
```

## Limitations

So far only Dynamo DB storage had been tested/used by the author.
