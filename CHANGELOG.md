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
