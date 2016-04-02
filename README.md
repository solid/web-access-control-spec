# Web Access Control (WAC)
[![](https://img.shields.io/badge/project-Solid-7C4DFF.svg?style=flat-square)](https://github.com/solid/solid)

Web Access Control (WAC) specification (as used by the Solid project). It
is based on Tim Berners-Lee's May 2009 proposal, as originally captured, and
subsequently evolved by the community, at
[Web Access Control Wiki](https://www.w3.org/wiki/WebAccessControl). This spec
is a particular subset of the options and extensions described in the wiki.

**Current Spec version:** `v.0.1.0` (see [CHANGELOG.md](CHANGELOG.md))

## Table of Contents

1. [Overview](#overview)
2. [Vocabulary](#vocabulary)
3. [Example WAC Document](#example-wac-document)
4. [Describing Agents](#describing-agents)
5. [Referring to Resources](#referring-to-resources)
6. [Modes of Access](#modes-of-access)

## Overview

Web Access Control (WAC) is a decentralized cross-domain access control system.
The main concepts should be familiar to developers, as they are similar to
access control schemes used in many file systems. It's concerned with giving
access to agents (users, groups and more) to perform various kinds of operations
(read, write, append, etc) on resources. WAC has several key features:

1. The resources are identified by URLs, and can refer to any web documents or
  resources.
2. It is *declarative* -- access control policies live in regular web documents,
  as opposed to in separate privileged subsystems (as they do in many operating
  systems).
3. Users and groups are also identified by URLs (specifically, by
  [Web IDs](https://github.com/solid/solid-spec#identity))
4. It is *cross-domain* -- all of its components, such as resources, agent Web
  IDs, and even the documents containing the access control policies, can
  potentially reside on separate domains. In other words, you can give access
  to a resource on one site to users and groups hosted on another site.

WAC documents contain a number of *Authorizations*, in Linked Data format
([Turtle](http://www.w3.org/TR/turtle/) by default, but also available in other
serializations). Each Authorization has statements describing: who the
authorized *agents* are, which *resources* they have access to, and what types
(*modes*) of access they are being granted.

Note: A familiarity with Linked Data and
[RDF Concepts](http://www.w3.org/TR/rdf11-concepts/) helps with understanding
the terminology used in this spec.

## Vocabulary

WAC uses the http://www.w3.org/ns/auth/acl ontology for its terms. Through the
rest of the spec, the prefix `acl:` is assumed to mean
`@prefix acl: <http://www.w3.org/ns/auth/acl#> .`

## Example WAC Document

Below is an example WAC document (often referred to as an 'ACL resource') that
specifies that Alice (as identified by her Web ID
`https://alice.databox.me/profile/card#me`) has full access (Read, Write and
Control) to one of her web resources, located at
`https://alice.databox.me/file1`.

The exact location of `file1`'s corresponding ACL resource is
technically opaque (the naming convention may differ between various servers
and implementations). In this example, for simplicity, we use the current
[LDNode](https://github.com/linkeddata/ldnode) convention and locate it
at `file1.acl`:

```ttl
# Contents of https://alice.databox.me/file1.acl
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

<#authorization1>
    a acl:Authorization;
    acl:agent <https://alice.databox.me/profile/card#me>;  # Alice's Web ID
    acl:accessTo <https://alice.databox.me/file1>;
    acl:mode
        acl:Read, acl:Write, acl:Control.
```

## Describing Agents

In WAC, we use the term *Agent* to identify *who* is allowed access to various
resources. In general, it is assumed to mean "someone or something that can
be referenced with a Web ID", which covers users, groups (as well as companies
and organizations), and software agents such as applications or services.

### Singular Agent

An authorization may list any number of individual agents (that are being given
access) by using the `acl:agent` predicate, and using their Web ID URIs as
subjects.

### Groups of Agents

If you need to give access to a particular group of agents, you can instead use
the `acl:agentClass` predicate, and point it to a resource which lists the
Web IDs of the individual members of that group.

#### All Agents (Public)

To specify that you're giving a particular mode of access to *everyone*
(for example, that your Web ID Profile is public-readable), you can use
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
quickly realize thats this recursion has to end somewhere.

As mentioned in the Introduction, one of the design goals of Web Access Control
is that the ACL resources themselves do not get any special treatment by
the server -- they are plain Web resources with Linked Data statements in them,
and so their own access has to be controlled by other Linked Data statements
that have to live somewhere. To avoid multiple levels of recursion, it is
recommended that the authorizations that control who has access to an ACL
document are placed *in the ACL document itself*.

In other words, an ACL document typically controls its own access, explicitly.
To extend a [previous example](#example-wac-document):

```ttl
# Contents of https://alice.databox.me/file1.acl
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

<#authorization1>
    # ... statements controlling access to file1

# This authorization concerns the ACL resource itself
<#authorization2>
    a acl:Authorization;
    acl:accessTo <>;  # gives access to this document
    acl:agent <https://alice.databox.me/profile/card#me>;  # to Alice's Web ID
    acl:mode
        acl:Read, acl:Write, acl:Control.
```

## Modes of Access

The `acl:mode` predicate denotes a class of operations that the agents can
perform on a resource.

##### `acl:Read`
gives access to a class of operations that can be described as "Read
Access". In a typical REST API (such as the one used by Solid), this includes
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
is a special-case access mode that gives an agent the ability to *modify the
ACL of a resource*. Note that it doesn't automatically imply that the agent
has `acl:Write` access to the resource itself, just to its corresponding ACL
document. For example, a resource owner may disable their own Write access
(to prevent accidental over-writing of a resource by an app), but be able to
change their access levels at a later point (since they retain `acl:Control`
access).
