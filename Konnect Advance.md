# Konnect Advance Team Organization and Access Control
This document outlines a new method for Konnect administrators to manage access to Konnect resources. Primarily, individual users can be grouped into teams and individual teams will have permissions to access certain runtimes and services in Konnect.
___

## Today's State
Today, a Konnect organization is set up as follows:

#### Runtimes (Resource)
- A single **Runtime Manager** lists all runtimes configured in an organization.
- All runtimes created within an organization are homogeneous.

#### Services (Resource)
- A single **Service Hub** catalog displays all services registered in an organization.

#### User (Subject)
- There is a flat user list
    - Each user can be assigned one or more roles
- Each role grants the user permission to configure some resource:
    - Organization Admin: Manage all objects, users and roles in the organization.
    - Service Admin: Create, edit and delete services and manage global configs.
    - Service Developer: Create and manage versions of existing services and their specs.
    - Service Page Editor: Manage the documentation and specs for services.
    - Portal Admin: Manage all developer portal content.
    - Runtime Admin: Manage runtimes and manage global configs.

### Problem Statement
> Enterprise organizations would like more fine grained control over what resources users in Konnect have access to and be able to manage these controls at scale.
___

## Users -> Teams (Subject)
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

## Runtimes -> Runtime Groups (Resource)
In order to gain fine grained control of runtime management access, runtimes need to be classified into **Runtime Groups**. As a result, previously homogeneous runtimes in an organization can be grouped into runtime groups classified in a multitude of ways.

Note: Runtime Groups will be an Enterprise tier feature only and therefore their usage limited to our Enterprise user base. Of this user base, we expect that a single organization will have on average around 6-12 runtime groups and no more than 100 or so at max.

#### Runtime Group Attributes
```
- Runtime Group Name (Unique ID)
- List of Runtimes
```



## Service (Resource)
- Service Name (Unique ID)






# KRNs (Kong Resource Names)
- KRNs are permissions that can be assigned to either a User or a Team

### Format
`region`: `reg/us`, `reg/emea`, `reg/apac`

`organization`: `org/org1`, `org/org2`, `org/org3`

`resourceGroup`: `runtime-group/prod`, `runtime-group/dev`, `services/frontend`, `services/backend`

`resource`: `runtime`, `service`

`verb`: `create`, `read`, `update`, `delete`

```
krn:region:organization:resourceGroup:resource!verb
```

### Examples
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

## Enterprise Example:
