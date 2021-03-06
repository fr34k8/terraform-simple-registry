# Terraform Simple Registry

This repository contains simple implementations of the Terraform registry
protocols.

This package is designed with the principle of "small pieces loosely joined":
it deals only with the Terraform protocols and expects other concerns to be
dealt with via complementary software. For example:

* Each of the registry services is provided as a separate program, allowing the
  user to decide which to use and how to deploy them.

* Authentication is not directly integrated, but can be applied by e.g. putting
  nginx in front of this program's server and configuring it to verify
  _JSON Web Tokens_, or similar.

* These programs provide read-only access to data available in the local
  filesystem. This data can either be placed manually or updated automatically
  by separate software in response to hooks from a remote Git repository, etc.

* A simple, static configuration system is used, so these files can either be
  hand-edited or generated automatically by some other external program.

Delegating all but the core concerns to other software gives users the
flexibility to customize their deployment and integrate these programs with
other systems and processes.

This program is not a HashiCorp product and is not directly supported by
the Terraform team at HashiCorp.

An "out-of-the-box" private registry solution, with support from HashiCorp,
is available as part of [Terraform Enterprise](https://www.hashicorp.com/products/terraform).
This implementation integrates well with other Terraform Enterprise
features.

## Included Servers

The design of this codebase is to have a separate server program for each of
the separate Terraform native service protocols it supports. The protocols
currently supported are:

* [Module registry v1](./cmd/terraform-modules-v1-server) (`modules.v1`)

## Service Discovery

Terraform uses a simple service discovery protocol to locate remote services
when given a "friendly hostname" by the user. For example, the hostname at
the start of a module source string like `example.com/foo/bar/baz` is such
a hostname, causing Terraform to attempt service discovery on that host.

The programs in this package each expose functionality for one of Terraform's
supported services. It's your responsibility as the deployer to place a
service discovery document on some suitable hostname to allow Terraform to
find the services you've deployed. It is not required that the servers
themselves be deployed at the same hostname.

Terraform looks for a discovery document by accessing the given hostname with
HTTPS and requesting the path `/.well-known/terraform.json`, at which it
expects to find an `application/json` response with content like the following:

```json
{
  "modules.v1": "http://modules.example.com/v1/"
}
```

Each key in this object relates to a single service, which in turn maps to one
server program in this repository. The documentation for each of the included
servers (linked above) defines which service discovery key it relates to.
