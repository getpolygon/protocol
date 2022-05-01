# The Polygon Protocol

Polygon is an effort to create a **fast**, **private**, **scalable** and **open** protocol for creating [federated social networks]. This paper defines the standards of the Polygon protocol and how they must be implemented and includes thorough recommendations for developers who are interested in implementing it themselves.

[federated social networks]: https://en.wikipedia.org/wiki/Distributed_social_network

## Table of Contents

- [Why?](#why)
- [Conformance](#conformance)
- [Recommended Software](#required-software)
  - [Database](#database)
  - [Blob Storage](#blob-storage)
  - [KV Database, PubSub, and Caching](#kv-database-pubsub-and-caching)
- [Optional Integrations](#optional-integrations)
- [Unique Identifiers](#unique-identifiers)
- [Username Specification](#username-specification)

## Why?

Centralization has helped onboard billions of people to the internet and created the stable, robust infrastructure on which it lives. At the same time, a handful of centralized entities have a stronghold on large swathes of the internet, unilaterally deciding what should and should not be allowed. - [ethereum.org](https://ethereum.org/en/web3/)

Consumers conduct much of their lives on the internet, yet few understand the critical issue of privacy and how their personal information is used, collected and shared by businesses. Your data can be stored indefinitely and used in both beneficial and unwelcome ways. - [security.berkeley.edu](https://security.berkeley.edu/news/why-should-we-care-about-online-privacy#:~:text=Consumers%20conduct%20much%20of%20their,both%20beneficial%20and%20unwelcome%20ways.)

Social media companies like Facebook and Instagram make a significant portion of their revenue by selling users’ data to data collection companies, who then sell it to other third parties for targeted advertisements. That explains why, once you like something on Facebook, you may see ads pop up for it everywhere you go online. - [security.org](https://www.security.org/blog/how-much-would-you-sell-your-social-media-data-for/#:~:text=You%20may%20not%20know%20this,third%20parties%20for%20targeted%20advertisements.)

A similar protocol, [ActivityPub] which has been implemented by [many projects], _can be [highly inefficient]_, and can be a hassle to operate in case of [DDoS attacks]. This, of course, impacts the overall performance of the instance, the resource usage, users' experience, and the fee of servicing your VPS.

[activitypub]: https://www.w3.org/TR/activitypub/
[highly inefficient]: https://feddit.de/post/13231
[many projects]: https://github.com/BasixKOR/awesome-activitypub
[ddos attacks]: https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/

## Conformance

As well as sections marked as non-normative, all authoring guidelines, diagrams, examples, and notes in this specification are non-normative. Everything else in this specification is normative.

The keywords **MAY**, **MUST**, **MUST NOT**, **SHOULD**, and **SHOULD NOT** are to be interpreted as described in [RFC2119].

[rfc2119]: https://tools.ietf.org/html/rfc2119/

## Recommended Software

Polygon is an open protocol, however, to ensure a good user and developer experience, implementers are _advised_ to use the tools officially recommended by the maintainers.

### Database

For a database, we recommend using [CockroachDB], which is a modern distributed SQL database designed for speed, scale, and survival. CockroachDB supports the [PostgreSQL wire protocol], which means that any driver that supports [PostgreSQL], can support CockroachDB. If you do not want to use CockroachDB, then you should use PostgreSQL or equivalent **SQL database**, however, scaling your instance might be a difficult problem, especially if you expect high loads on your instance.

> You can consult [this guide](https://www.cockroachlabs.com/docs/stable/postgresql-compatibility.html#drop-primary-key) to check which features of PostgreSQL are supported by CockroachDB and which are not.

Implementers are **discouraged** to use _unstructured, document databases_ such as [MongoDB], despite faster performance and [easier API][mongodb-query-api]. Data stored and used by the Polygon protocol is highly structured and contains logic, which would otherwise be hard and time-consuming from developers' perspective using a [document-oriented database].

[mongodb]: https://www.mongodb.com/
[postgresql]: https://postgresql.org/
[cockroachdb]: https://github.com/cockroachdb/cockroach/
[postgresql wire protocol]: https://www.cockroachlabs.com/docs/v21.2/architecture/sql-layer.html#postgresql-wire-protocol
[mongodb-query-api]: https://www.mongodb.com/docs/manual/tutorial/query-documents/
[document-oriented database]: https://en.wikipedia.org/wiki/Document-oriented_database/

### Blob Storage

For blob storage, we recommend using [MinIO], which is software-defined high performance object storage released under [GNU Affero General Public License v3.0][gpl3]. MinIO provides an Amazon Web Services S3-compatible API and closely follows all the standards defined in the [AWS S3 specification]. MinIO is distributed, thus, it is easy to scale. There are other alternatives which support the S3 protocol too, such as [LocalStack] and [SeaweedFS].

[minio]: https://min.io
[gpl3]: https://www.gnu.org/licenses/agpl-3.0.en.html
[seaweedfs]: https://github.com/chrislusf/seaweedfs#quick-start
[localstack]: https://github.com/chrislusf/seaweedfs#quick-start
[aws s3 specification]: https://docs.aws.amazon.com/s3/index.html

### KV Database, PubSub, and Caching

For a message-broker, cache and key-value database, we recommend using [Redis], which is an in-memory data structure store, used as a distributed, in-memory key–value database, cache and message broker, with optional durability. We recommend using it, since its usage is not limited only to brokering messages, like [RabbitMQ]. Both Redis and RabbitMQ support clustering, thus, scaling them should not be an issue.

[redis]: https://redis.io/
[rabbitmq]: https://www.rabbitmq.com/

## Optional Integrations

To make it easier for developers to monitor resource usage and debug their implementations using the Polygon Protocol and report possible bugs, we _optionally_ recommend integrating [Prometheus] and [Grafana] to your software, only for _**internal**_ use.

[Learn more about securing Grafana] <> [Learn more about securing Prometheus]

[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/
[learn more about securing grafana]: https://grafana.com/docs/grafana/latest/administration/security/
[learn more about securing prometheus]: https://prometheus.io/docs/operating/security/

## Unique Identifiers

All models defined in the protocol, **must** only use [ULID] as their ID. Implementers **must not** use other UID generators, such as [UUID]. The protocol uses ULID, because they are universally unique, lexicographically sortable identifiers, and are smaller in length when compared to UUIDs.

Suppose that we want to fetch all posts of a user, after a certain post ID. To do that with UUID, we would have to write this query:

```sql
SELECT *
  FROM posts
 WHERE username = '<insert_username>'
   AND (created_at < '<insert_date>' OR (created_at = '<insert_date>' AND p.id < '<insert_uuid>'));
```

Whereas with ULID, this is very straightforward:

```sql
SELECT * FROM posts WHERE id > '<insert_ulid>' AND username = '<insert_username>';
```

Besides being smaller in length, ULID eliminates the need to have separate fields in our tables to track the creation time of an object, such as `created_at`.

From the official ULID documentation, ULID's benefits over UUID include:

- 128-bit compatibility with UUID
- 1.21e+24 unique ULIDs per millisecond
- Lexicographically sortable!
- Canonically encoded as a 26 character string, as opposed to the 36 character UUID
- Uses Crockford's base32 for better efficiency and readability (5 bits per character)
- Case insensitive
- No special characters (URL safe)

[ulid]: https://github.com/ulid/spec
[uuid]: https://en.wikipedia.org/wiki/Universally_unique_identifier

We recommend creating a custom SQL domain, for better SQL code readability. Since, a ULID consists of 26 characters the domain creation query would look like this:

```sql
CREATE DOMAIN ulid AS CHAR(26);
```

PostgreSQL does not support ULID out of the box, thus, implementers must create a custom PLpgSQL function to generate ULIDs. You can find a pre-made function at [getpolygon/pgulid] repository on GitHub, and add it to your SQL migrations.

After successfully adding the function, you can use it to create tables in the following way:

```sql
CREATE TABLE foo (
  id ulid DEFAULT gen_ulid() PRIMARY KEY
);
```

It is also worth mentioning that adding an ID to every model is optional. Furthermore, adding an ID to a model that does not necessarily need one is **highly discouraged**. Using our recommended approach will save disk and memory resources.

[getpolygon/pgulid]: https://github.com/getpolygon/pgulid

## Username Specification

All protocol implementations _**must**_ adhere the following specification for usernames:

```bash
username:domain.tld
username:192.0.2.146
username:subdomain.domain.tld
username:2001:0db8:85a3:0000:0000:8a2e:0370:7334 # IPv6 address
```

We are not limiting the instance addresses of the users only to their domains, since some of them might not want to buy or maintain a [fully qualified domain name](fqdn). Users are free to host their instances wherever they want.

[fqdn]: https://en.wikipedia.org/wiki/Fully_qualified_domain_name

Usernames can be matched with the special RegEx expression, created specifically for the Polygon Protocol, below:

```regex
((?:[\w][\.]{0,1})*[\w]):((([a-z0-9|-]+\.)*[a-z0-9|-]+\.[a-z]+)|(\b(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[1-9])\.)(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\.){2}(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\b)|((([0-9a-fA-F]{1,4}:){7,7}[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,7}:|([0-9a-fA-F]{1,4}:){1,6}:[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,5}(:[0-9a-fA-F]{1,4}){1,2}|([0-9a-fA-F]{1,4}:){1,4}(:[0-9a-fA-F]{1,4}){1,3}|([0-9a-fA-F]{1,4}:){1,3}(:[0-9a-fA-F]{1,4}){1,4}|([0-9a-fA-F]{1,4}:){1,2}(:[0-9a-fA-F]{1,4}){1,5}|[0-9a-fA-F]{1,4}:((:[0-9a-fA-F]{1,4}){1,6})|:((:[0-9a-fA-F]{1,4}){1,7}|:)|fe80:(:[0-9a-fA-F]{0,4}){0,4}%[0-9a-zA-Z]{1,}|::(ffff(:0{1,4}){0,1}:){0,1}((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])|([0-9a-fA-F]{1,4}:){1,4}:((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9]))))
```

### Description of the RegEx

- `((?:[\w][\.]{0,1})*[\w])` - Will match the `username`. Is URL safe. More information about the pattern can be found in the following StackOverflow thread: <https://stackoverflow.com/a/68642388/14575901>

- `(([a-z0-9|-]+\.)*[a-z0-9|-]+\.[a-z]+)` - Will match any RFC-compliant domain name, with the subdomain, if specified. More information about this pattern can be found in the following StackOverflow thread: <https://stackoverflow.com/a/8959842/14575901>

- `(\b(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[1-9])\.)(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\.){2}(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\b)` - Will match only a valid IPv4 address. More information about this pattern can be found in the following StackOverflow thread: <https://stackoverflow.com/a/69942902/14575901>

- `(([0-9a-fA-F]{1,4}:){7,7}[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,7}:|([0-9a-fA-F]{1,4}:){1,6}:[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,5}(:[0-9a-fA-F]{1,4}){1,2}|([0-9a-fA-F]{1,4}:){1,4}(:[0-9a-fA-F]{1,4}){1,3}|([0-9a-fA-F]{1,4}:){1,3}(:[0-9a-fA-F]{1,4}){1,4}|([0-9a-fA-F]{1,4}:){1,2}(:[0-9a-fA-F]{1,4}){1,5}|[0-9a-fA-F]{1,4}:((:[0-9a-fA-F]{1,4}){1,6})|:((:[0-9a-fA-F]{1,4}){1,7}|:)|fe80:(:[0-9a-fA-F]{0,4}){0,4}%[0-9a-zA-Z]{1,}|::(ffff(:0{1,4}){0,1}:){0,1}((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])|([0-9a-fA-F]{1,4}:){1,4}:((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9]))` - Will match only a valid IPv6 address. More information about this pattern can be found in the following StackOverflow thread: <https://stackoverflow.com/a/17871737/14575901>

The archived versions of the threads mentioned earlier can be found at

- [IPv4 RegEx pattern archive](https://web.archive.org/web/20220427104242/https://stackoverflow.com/questions/5284147/validating-ipv4-addresses-with-regexp/69942902)
- [IPv6 RegEx pattern archive](https://web.archive.org/web/20220427104010/https://stackoverflow.com/questions/53497/regular-expression-that-matches-valid-ipv6-addresses/17871737)
- [URL-safe username RegEx pattern archive](https://web.archive.org/web/20191219020147/https://stackoverflow.com/questions/32543090/instagram-username-regex-php)
- [RFC-Compliant domain RegEx pattern archive](https://web.archive.org/web/20220427104531/https://stackoverflow.com/questions/8959765/need-regex-to-get-domain-subdomain/8959842)
