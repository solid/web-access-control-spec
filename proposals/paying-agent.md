### The 'Paying Agent' agent class

**Note:** This proposal is still experimental, and clients cannot rely
on it yet. See the [Test Suite report](https://github.com/solid/test-suite#table)
for the latest information on which servers currently support it.

The current WAC spec defines two agent classes:
* `foaf:Agent` (allows access to any agent, i.e., the public)
* `acl:AuthenticatedAgent` (allows access to any authenticated agent)

This proposal adds a third one:
* `acl:AuthenticatedAgent` (allows access to any paying agent)

For more information about how to use this agent class in combination with
the `402 Payment Required` http response code and the `Pay` http response header,
and about how to determine whether a given WebID
is a paying agent for a given resource, see [Solid Webmonetization](https://github.com/solid/webmonetization/blob/main/README.md#requiring-payment-for-resources).