### Groups of Agents and `agentClass`

**Note:** Current spec uses `acl:agentGroup` and `vcard:hasMember` to implement
group membership. This section is saved here for reference / archival purposes.

To give access to a group of agents, use the `acl:agentClass` predicate.
The object of an `agentClass` statement is a hash fragment identifier that
resolves to an RDF class statement in a **Group Listing** document.
If a WebID is listed in that document, *and* it's of the specified class, it is
given access.

Example ACL resource, `shared-file1.acl`, containing a group permission:

```ttl
# Contents of https://alice.databox.me/docs/shared-file1.acl
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

# Individual authorization - Alice has Read/Write/Control access
<#authorization1>
    a acl:Authorization;
    acl:accessTo <https://alice.example.com/docs/shared-file1>;
    acl:mode acl:Read, acl:Write, acl:Control;
    acl:agent <https://alice.example.com/profile/card#me>.

# Group authorization, giving Read/Write access to two groups, which are
# specified in the 'work-groups' document.
<#authorization2>
    a acl:Authorization;
    acl:accessTo <https://alice.example.com/docs/shared-file1>;
    acl:mode acl:Read, acl:Write;
    acl:agentClass <https://alice.example.com/work-groups#Accounting>;
    acl:agentClass <https://alice.example.com/work-groups#Management>.
```

Corresponding `work-groups` Group Listing document:

```ttl
# Contents of https://alice.example.com/work-groups
@prefix acl: <http://www.w3.org/ns/auth/acl#>.
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>.

<> a acl:GroupListing.

<#Employee> a rdfs:Class.
<#Accounting> a rdfs:Class.
<#Management> a rdfs:Class.

<https://bob.example.com/profile/card#me> a <#Employee>, <#Accounting>.
<https://candice.example.com/profile/card#me> a <#Employee>, <#Accounting>.

<https://deb.example.com/profile/card#me> a <#Employee>, <#Management>.
```
