- [Konnect Advance Team Organization and Access Control](#konnect-advance-team-organization-and-access-control)
  - [Today's State](#todays-state)
      - [User](#user)
      - [Runtimes](#runtimes)
      - [Services](#services)
  - [Problem Statement](#problem-statement)
  - [Proposal](#proposal)
    - [Users -> Teams](#users---teams)
    - [Runtimes -> Runtime Groups](#runtimes---runtime-groups)
    - [Service](#service)
    - [KRNs (Kong Resource Names)](#krns-kong-resource-names)
- [Enterprise Example (TBD)](#enterprise-example)

___
# Konnect Advance Team Organization and Access Control
This document outlines a new method for Konnect administrators to manage access to Konnect resources. Primarily, individual users can be grouped into teams and individual teams will have permissions to access certain runtimes and services in Konnect.

## Today's State
Today, a Konnect organization is set up as follows:

#### User
- There is a flat user list with linear hierarchy.
    - Each user can be assigned one or more roles
- Each role grants the user permission to configure some resource:
    - Organization Admin: Manage all objects, users and roles in the organization.
    - Service Admin: Create, edit and delete services and manage global configs.
    - Service Developer: Create and manage versions of existing services and their specs.
    - Service Page Editor: Manage the documentation and specs for services.
    - Portal Admin: Manage all developer portal content.
    - Runtime Admin: Manage runtimes and manage global configs.

#### Runtimes
- A single **Runtime Manager** lists all runtimes configured in an organization.
- All runtimes created within an organization are homogeneous.

#### Services
- A single **Service Hub** catalog displays all services registered in an organization.

### Problem Statement
> Enterprise organizations would like more fine grained control over what resources users in Konnect have access to and be able to manage these controls at scale.
___

## Proposal
### Users -> Teams
In order to manage multiple users and their permissions at scale, individual users need to be grouped into **teams**. Like users, teams can be assigned one or more permissions and all users in the team will be granted the permissions assigned to the team. 

#### Team Attributes
```
- Kong Team Name (Unique ID)
- List of Users
- List of Permissions
```

Furthermore, since enterprise Konnect users may already belong to functional teams managed by an identity provider (IdP). It should be possible to map IdP groups to Konnect teams such that members of IdP groups are granted the permissions assigned to the team the group is associated with.

#### IdP Mapping Attributes
```
- IdP Group Name (Unique ID)
- Kong Team Name (Unique ID)
```
TBD: If a user is in multiple teams, is it possible there might be a permission conflict?

### Runtimes -> Runtime Groups
In order to gain fine grained control of runtime management access, runtimes need to be classified into **Runtime Groups**. As a result, previously homogeneous runtimes in an organization can be grouped into runtime groups classified in a multitude of ways.

Note: Runtime Groups will be an Enterprise tier feature only and therefore their usage limited to our Enterprise user base. Of this user base, we expect that a single organization will have on average around 6-12 runtime groups and no more than 100 or so at max.

#### Runtime Group Attributes
```
- Runtime Group Name (Unique ID)
- List of Runtimes
```

### Service
Unlike runtimes, services are already non-homogeneous so further classification for fine grained control is not necessary. Permissions to access each service can be assigned directly to a user or team.
#### Service Attributes
```
- Service Name (Unique ID)
```

### KRNs (Kong Resource Names)
In order to identify what resources a user or team is authorized to access, a standardized **KRN** convention will be used to grant permissions. KRN will map an action to a resource/resources which can be assigned to either a user or team. The team will then be granted access to resources specified by the list of KRN permissions.

#### Format
```
krn:region:organization:resourceGroup:resource!verb
```

| Field | Definition | Example |
|:--|:--|:--|
| region | Regional Identification | `reg/us`, `reg/emea`, `reg/apac` |
| organization | Organization Name | `org/org1`, `org/org2`, `org/org3` |
| resourcePath | Path of the resources to grant access to | `runtime-group/prod`, `runtime-group/dev`, `services/frontend`, `services/backend` |
| resourceIds | Ids of resource grant access to | `runtime`, `service` |
| verb | Action allowed | `create`, `read`, `update`, `delete` |


#### Examples
```python
# Runtime Group KRNs
- krn:reg/us:org/org1:runtime-group/prod:runtime!create
- krn:reg/us:org/org1:runtime-group/prod:runtime/*!read
- krn:reg/us:org/org1:runtime-group/prod:runtime/*!update
- krn:reg/us:org/org1:runtime-group/prod:runtime/*!delete

# Service KRNs
- krn:reg/us:org/org1:services/frontend:service!create
- krn:reg/us:org/org1:services/frontend:service/*!read
- krn:reg/us:org/org1:services/frontend:service/*!update
- krn:reg/us:org/org1:services/frontend:service/*!delete
```

___

# Enterprise Example:
TBD
