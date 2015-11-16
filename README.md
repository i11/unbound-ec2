# unbound-ec2

[![Build Status](https://api.travis-ci.org/SimpleFinance/unbound-ec2.svg)](https://travis-ci.org/SimpleFinance/unbound-ec2)
[![Version](http://img.shields.io/pypi/v/unbound-ec2.svg?style=flat)](https://pypi.python.org/pypi/unbound-ec2/)

This module uses the [Unbound](http://unbound.net) DNS resolver to answer simple DNS queries using EC2 API calls.
For example, the following query would match an EC2 instance with a `Name` tag of `foo.example.com`:

```
$ dig -p 5003 @127.0.0.1 foo.dev.example.com
[1380410835] unbound[25204:0] info: unbound_ec2: handling forward query for foo.dev.example.com.

; <<>> DiG 9.8.1-P1 <<>> -p 5003 @127.0.0.1 foo.dev.example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5696
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;foo.dev.example.com.	IN	A

;; ANSWER SECTION:
foo.dev.example.com. 300 IN	A	10.0.0.2
foo.dev.example.com. 300 IN	A	10.0.0.1

;; Query time: 81 msec
;; SERVER: 127.0.0.1#5003(127.0.0.1)
;; WHEN: Sat Sep 28 23:27:16 2013
;; MSG SIZE  rcvd: 77
```

## Installation

On Debian family, install the `unbound`, `python-unbound` system packages.
On Redhat family, install the `unbound`, `unbound+python` system packages.

Then, install `unbound-ec2`:
```
pip install unbound-ec2
```

## Configuration

The following settings must be added to your Unbound configuration:

```
server:
    chroot: ""
    module-config: "validator python iterator"

python:
    python-script: "/etc/unbound/unbound_ec2_script"
```

EC2 module can be configured by specifying values in /etc/unbound/unbound_ec2.conf or setting environment variables in
`/etc/default/unbound`.

See unbound_ec2.conf.example and default_unbound.example for more information.

You can also define `AWS_ACCESS_KEY` and `AWS_SECRET_ACCESS_KEY` entries in the environment directory.
When `unbound-ec2` is run on an EC2 instance, though, it will automatically use an IAM instance profile if one is available.

## Considerations

`unbound-ec2` queries the EC2 API to answer requests about names inside the specified `zone`.
All other requests are handled normally by Unbound's caching resolver if caching type server was chosen.

For requests for names within the specified `zone`, `unbound_ec2` calls
[`DescribeInstances`](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-DescribeInstances.html)
and filters the results using defined lookup filters (default is instances in the `running` state).

When more than one instance matches the `DescribeInstances` query, `unbound-ec2` will return multiple A records in a round-robin. 
In case of caching type server, query results will be cached by Unbound, and a TTL (default: 300 seconds) is defined
to encourage well-behaved clients to cache the information themselves.

Public addresses, IPv6, and reverse DNS lookups (PTR) are not yet supported.

## Unit tests

Run with

```
python setup.py test
```

## License

See LICENSE file.
