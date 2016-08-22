##### v.0.4.0

- Change Group ACL definition spec to one based on `vcard:hasMember` instead of
  `acl:agentClass`. Move previous `agentClass` based section to `proposals/` for
  archival purposes.

##### v.0.3.1

- Add a discussion of infinite loops in Group ACL resolution

##### v.0.3.0

- Expand the agentClass / Group acl definition, clarifying usage of
  `acl:agentClass`, discussing Group Listing documents

##### v.0.2.0

- Add 'Access Control List Resources' sections, discuss individual vs
  inherited ACLs.
- Discuss the `acl` link relation and ACL discovery
- Add a section on the ACL Inheritance Algorithm
- Move the Vocabulary section into the 'Representation Format' section
- Add a section for `acl:defaultForNew` ('Default (Inherited) Authorizations')
- Add a 'Not Supported by Design' section
- Added a side-note about how `acl:defaultForNew` will soon be renamed to
  `acl:default`

##### v0.1.1

- Change the [Referring to Resources](#referring-to-resources) section --
  the recommended solution is simply to use `acl:Control`, and not refer
  to the ACL resource itself in its own contents.
- Remove the phrasing that WAC does not privilege the ACL resources.
- Clarify that `acl:Control` gives access even to *read* the ACL resource.
  Without that permission, a user cannot read the resource.

##### v0.1.0

First draft of spec, as of Apr 2016.
