# Web Access Control (WAC)
[![](https://img.shields.io/badge/project-Solid-7C4DFF.svg?style=flat-square)](https://github.com/solid/solid)

Web Access Control (WAC) specification (as used by the Solid project). It
is based on Tim Berners-Lee's May 2009 proposal, as originally captured, and
subsequently evolved by the community, at
[Web Access Control Wiki](https://www.w3.org/wiki/WebAccessControl). This spec
is a particular subset of the options and extensions described in the wiki.

**Current Spec version:** `v.0.2.0` (see [CHANGELOG.md](CHANGELOG.md))

## Table of Contents

1. [Overview](#overview)
2. [Access Control List Resources](#access-control-list-resources)
  * [Containers and Inherited ACLs](#containers-and-inherited-acls)
  * [Individual Resource ACLs](#individual-resource-acls)
  * [ACL Resource Location Discovery](#acl-resource-location-discovery)
3. [ACL Inheritance Algorithm](#acl-inheritance-algorithm)
4. [Representation Format](#representation-format)
5. [Example WAC Document](#example-wac-document)
6. [Describing Agents](#describing-agents)
7. [Referring to Resources](#referring-to-resources)
8. [Modes of Access](#modes-of-access)
9. [Default (Inherited) Authorizations](#default-inherited-authorizations)
10. [Not Supported by Design](#not-supported-by-design)

## Overview

Web Access Control (WAC) is a decentralized cross-domain access control system.
The main concepts should be familiar to developers, as they are similar to
access control schemes used in many file systems. It's concerned with giving
access to agents (users, groups and more) to perform various kinds of operations
(read, write, append, etc) on resources. WAC has several key features:

1. The resources are identified by URLs, and can refer to any web documents or
  resources.
2. It is *declarative* -- access control policies live in regular web documents,
  which can be exported/backed easily, using the same mechanism as you would
  for backing up the rest of your data.
3. Users and groups are also identified by URLs (specifically, by
  [WebIDs](https://github.com/solid/solid-spec#identity))
4. It is *cross-domain* -- all of its components, such as resources, agent
  WebIDs, and even the documents containing the access control policies, can
  potentially reside on separate domains. In other words, you can give access
  to a resource on one site to users and groups hosted on another site.

## Access Control List Resources

In a system that uses Web Access Control, each web resource has a set of
*Authorization* statements describing:

1. Who has access to that resource (that is, who the authorized *agents* are)
2. What types (or *modes*) of access they have

These Authorizations are either explicitly set for an individual resource, or
(more often) inherited from that resource's parent folder or container. In
either case, the Authorization statements are placed into separate WAC
documents called *Access Control List Resources* (or simply *ACLs*).

### Containers and Inherited ACLs

The WAC system assumes that web documents are placed in hierarchical containers
or folders. For convenience, users do not have to specify permissions on each
individual resource -- they can simply set permissions on a container, add a
[`acl:defaultForNew`](#default-inherited-authorizations) predicate, and have all
of the resources in that container [inherit](#acl-inheritance-algorithm) those
permissions.

### Individual Resource ACLs

For fine-grained control, users can specify a set of permissions for each
individual resource (which overrides any permissions of its parent container).
See the [Example WAC Document](#example-wac-document) section to get an idea
for what that would look like.

### ACL Resource Location Discovery

Given a URL for an individual resource or container, a client can discover the
location of its corresponding ACL by performing a `HEAD` (or a `GET`) request
and parsing the `rel="acl"` link relation.

Example request to discover the location of the ACL resource for a web document
at `http://example.org/docs/file1` is given below:

```http
HEAD /docs/file1 HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Link: <file1.acl>; rel="acl"
```

The request to discover the location of a container's ACL resource looks
similar:

```http
HEAD /docs/ HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Link: <.acl>; rel="acl"
```

Note that the `acl` link relation uses relative path URLs (the absolute URL of
the ACL resource in the above example would be `/docs/.acl`).

Clients MUST NOT assume that the location of an ACL resource can be
deterministically derived from a document's URL. For example, given a document
with a URL of `/docs/file1`, clients cannot rely on the assumption that an ACL
resource exists at `/docs/file1.acl`, simply using `.acl` as a prefix. The
actual naming convention for ACL resources can differ for each individual
implementation (or even for each individual server). If one server locates the
ACL resource by appending the suffix `.acl`, another server could place the ACL
resources into a sub-container (locating it at `/docs/.acl/file1.acl` for the
example above).

## ACL Inheritance Algorithm

The following algorithm is used by servers to determine which ACL resources
(and hence which set of Authorization statements) apply to any given resource:

1. Use the document's own ACL resource if it exists (in which case, stop here).
2. Otherwise, look for authorizations to inherit from the ACL of the document's
  container. If those are found, stop here.
3. Failing that, check the container's *parent* container to see if *that* has
  its own ACL file, and see if there are any permissions to inherit.
4. Failing that, move up the container hierarchy until you find a container
  with an existing ACL file, which has some permissions to inherit.
5. The *root container* of a user's account MUST have an ACL resource specified.
  (If all else fails, the search stops there.)

It is considered an anti-pattern for a *client* to perform those steps, however.
A client may [discover](#acl-resource-location-discovery) and load a document's
individual resource (for example, for the purposes of editing its permissions).
If such a resource does not exist, a client SHOULD NOT search for, or interact
with, the inherited ACLs from an upstream container.

### ACL Inheritance Algorithm Example

*Note:* The server in the examples below is using the ACL naming convention of
appending `.acl` to resource URLs, for simplicity. As mentioned in the section
on [ACL discovery](#acl-resource-location-discovery), clients should not use
or assume any naming convention.

A request (to read or write) has arrived for a document located at
`/documents/papers/paper1`. The server does as follows:

1. First, it checks for the existence of an individual corresponding ACL
  resource. (That is, it checks if `paper1.acl` exists.) If this individual ACL
  resource exists, the server uses the Authorization statements in that ACL. No
  other statements apply.
2. If no individual ACL exists, the server next checks to see if the
  `/documents/papers/` container (in which the document resides) has its own
  ACL resource (here, `/documents/papers/.acl`). If it finds that, the server
  reads each authorization in the container's ACL, and if any of them contain an
  `acl:defaultForNew` predicate, the server will use them (as if they were
  specified in `paper1.acl`). Again, if any such authorizations are found, the
  process stops there and no other statements apply.
3. If the document's container has no ACL resource of its own, the search
  continues upstream, in the *parent* container. The server would check if
  `/documents/.acl` exists, and then `/.acl`, until it finds some authorizations
  that contain `acl:defaultForNew`.
4. Since the root container (here, `/`) MUST have its own ACL resource, the
  server would use the authorizations there as a last resort.

See the [Default (Inherited) Authorizations](#default-inherited-authorizations)
section below for an example of what a container's ACL resource might look like.

## Representation Format

The permissions in an ACL resource are stored in Linked Data format
([Turtle](http://www.w3.org/TR/turtle/) by default, but also available in other
serializations).

*Note: A familiarity with Linked Data and
[RDF Concepts](http://www.w3.org/TR/rdf11-concepts/) helps with understanding
the terminology used in this spec.*

WAC uses the [`http://www.w3.org/ns/auth/acl`](http://www.w3.org/ns/auth/acl)
ontology for its terms. Through the rest of the spec, the prefix `acl:` is
assumed to mean `@prefix acl: <http://www.w3.org/ns/auth/acl#> .`

## Example WAC Document

Below is an example ACL resource that specifies that Alice (as identified by her
WebID `https://alice.databox.me/profile/card#me`) has full access (Read, Write
and Control) to one of her web resources, located at
`https://alice.databox.me/docs/file1`.

```ttl
# Contents of https://alice.databox.me/docs/file1.acl
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

<#authorization1>
    a acl:Authorization;
    acl:agent <https://alice.databox.me/profile/card#me>;  # Alice's WebID
    acl:accessTo <https://alice.databox.me/docs/file1>;
    acl:mode
        acl:Read, acl:Write, acl:Control.
```

## Describing Agents

In WAC, we use the term *Agent* to identify *who* is allowed access to various
resources. In general, it is assumed to mean "someone or something that can
be referenced with a WebID", which covers users, groups (as well as companies
and organizations), and software agents such as applications or services.

### Singular Agent

An authorization may list any number of individual agents (that are being given
access) by using the `acl:agent` predicate, and using their WebID URIs as
objects.

### Groups of Agents

If you need to give access to a particular group of agents, you can instead use
the `acl:agentClass` predicate, and point it to a resource which lists the
WebIDs of the individual members of that group.

#### All Agents (Public)

To specify that you're giving a particular mode of access to *everyone*
(for example, that your WebID Profile is public-readable), you can use
`acl:agentClass foaf:Agent` to denote that you're giving access to the class
of *all* agents (the general public). For example:

```ttl
@prefix acl: <http://www.w3.org/ns/auth/acl#>.
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

<#authorization2>
    a acl:Authorization;
    acl:agentClass foaf:Agent;  # everyone
    acl:mode acl:Read;  # has Read-only access
    acl:accessTo <https://alice.databox.me/profile/card>. # to the public profile
```

## Referring to Resources

The `acl:accessTo` predicate specifies *which resources* you're giving access
to, using their URLs as the subjects.

### Referring to the ACL Resource Itself

Since an ACL resource is a plain Web document in itself, what controls who
has access to *it*? While an ACL resource *could* in theory have its own
corresponding ACL document (for example, `file1.acl` controls access to `file1`,
and `file1.acl.acl` could potentially control access to `file1.acl`), one
quickly realizes thats this recursion has to end somewhere.

Instead, the [`acl:Control` access mode](#aclcontrol) is used (see below), to
specify who has access to alter (or even view) the ACL resource.

## Modes of Access

The `acl:mode` predicate denotes a class of operations that the agents can
perform on a resource.

##### `acl:Read`
gives access to a class of operations that can be described as "Read
Access". In a typical REST API (such as the one used by
[Solid](https://github.com/solid/solid-spec#https-rest-api)), this includes
access to HTTP verbs `GET`, and `HEAD`. This also includes any kind of
QUERY or SEARCH verbs, if supported.

##### `acl:Write`
gives access to a class of operations that can modify the resource. In a REST
API context, this would include `PUT`, `POST`, `DELETE` and `PATCH`. This also
includes the ability to perform SPARQL queries that perform updates, if those
are supported.

##### `acl:Append`
gives a more limited ability to write to a resource -- Append-Only. This
generally includes the HTTP verb `POST`, although some implementations may
also extend this mode to cover non-overwriting `PUT`s, as well as the
`INSERT`-only portion of SPARQL-based `PATCH`es. A typical example of Append
mode usage would be a user's Inbox -- other agents can write (append)
notifications to the inbox, but cannot alter or read existing ones.

##### `acl:Control`
is a special-case access mode that gives an agent the ability to *view and
modify the ACL of a resource*. Note that it doesn't automatically imply that the
agent has `acl:Read` or `acl:Write` access to the resource itself, just to its
corresponding ACL document. For example, a resource owner may disable their own
Write access (to prevent accidental over-writing of a resource by an app), but
be able to change their access levels at a later point (since they retain
`acl:Control` access).

## Default (Inherited) Authorizations

As previously mentioned, not every document needs its own individual ACL
resource and its own authorizations. Instead, one can can create an
Authorization for a container (in the container's own ACL resource), and then
use the `acl:defaultForNew` predicate to denote that any resource within that
container will *inherit* that authorization. To put it another way, if an
Authorization contains `acl:defaultForNew`, it will be applied *by default* to
any resource in that container.

You can override the default inherited authorization for any resource by
creating an individual ACL just for that resource.

An example ACL for a container would look something like:

```ttl
# Contents of https://alice.databox.me/docs/.acl
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

<#authorization1>
    a acl:Authorization;

    # These statements specify access rules for the /docs/ container itself:
    acl:agent <https://alice.databox.me/profile/card#me>;
    acl:accessTo <https://alice.databox.me/docs/>;
    acl:mode
        acl:Read, acl:Write, acl:Control;

    # defaultForNew says: this authorization (the statements above) will also
    #   be inherited by any resource within that container that doesn't have its
    #   own ACL.
    acl:defaultForNew <>.
```

**Note:** The `acl:defaultForNew` predicate will soon be renamed to
`acl:default`, both in the specs and in implementing servers. The semantics, as
described here, will remain the same

## Not Supported by Design

This section describes some features or acl-related terms that are not included
in this spec by design.

##### Resource Owners
WAC has no formal notion of a resource owner, and no explicit Owner access mode.
For convenience/UI purposes, it may be assumed that Owners are agents that have
Read, Write and Control permissions.

##### acl:accessToClass
The predicate `acl:accessToClass` is not supported.

##### Regular Expressions
The use of regular expressions in statements such as `acl:agentClass` is not
supported.
