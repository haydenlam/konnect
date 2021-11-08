- [Konnect Advance Team Organization and Access Control](#konnect-advance-team-organization-and-access-control)
  - [Today's State](#todays-state)
    - [Problem Statement](#problem-statement)
  - [Proposal](#proposal)
    - [Users -> Teams](#users---teams)
    - [Runtimes -> Runtime Groups](#runtimes---runtime-groups)
    - [Service](#service)
    - [KRNs (Kong Resource Names)](#krns-kong-resource-names)
- [Enterprise Customization Example:](#enterprise-customization-example)
    - [Scenario](#scenario)
    - [1. Creating Teams](#1-creating-teams)
    - [2. Setting Service Hub Permissions](#2-setting-service-hub-permissions)
    - [3. Setting Runtime Manager Permissions](#3-setting-runtime-manager-permissions)

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
| region | Regional Identification | `reg/{regionId}` |
| organization | Organization Name | `org/{orgId}` |
| resourcePath | Path of the resources to grant access to | `runtime-groups/{runtimeGroupId}`, `services/{serviceId}` |
| action | Action allowed | `create`, `read`, `update`, `delete` |


#### Examples
```YAML
# Runtime Group Permissions
Permissions:
  - id: xxxxxxxx-rtgp-1111-xxxx-xxxxxxxxxxxx
    name: runtime-group-create
    description: Permission to create Runtime Group
    resource: krn:reg/{region}:org/{orgID}:runtime-groups
    action: create
  - id: xxxxxxxx-rtgp-2222-xxxx-xxxxxxxxxxxx
    name: runtime-group-read
    description: Permission to read Runtime Group
    resource: krn:reg/{region}:org/{orgID}:runtime-groups/{id}
    action: read
  - id: xxxxxxxx-rtgp-3333-xxxx-xxxxxxxxxxxx
    name: runtime-group-update
    description: Permission to update Runtime Group
    resource: krn:reg/{region}:org/{orgID}:runtime-groups/{id}
    action: update
  - id: xxxxxxxx-rtgp-4444-xxxx-xxxxxxxxxxxx
    name: runtime-group-delete
    description: Permission to delete Runtime Group
    resource: krn:reg/{region}:org/{orgID}:runtime-groups/{id}
    action: delete
    
# Service Permissions
Permissions:
  - id: xxxxxxxx-svcp-1111-xxxx-xxxxxxxxxxxx
    name: services-create
    description: Permission to create Services
    resource: krn:reg/{region}:org/{orgID}:services
    action: create
  - id: xxxxxxxx-svcp-2222-xxxx-xxxxxxxxxxxx
    name: services-read
    description: Permission to read Services
    resource: krn:reg/{region}:org/{orgID}:services/{id}
    action: read
  - id: xxxxxxxx-svcp-3333-xxxx-xxxxxxxxxxxx
    name: services-update
    description: Permission to update Services
    resource: krn:reg/{region}:org/{orgID}:services/{id}
    action: update
  - id: xxxxxxxx-svcp-4444-xxxx-xxxxxxxxxxxx
    name: services-delete
    description: Permission to delete Services
    resource: krn:reg/{region}:org/{orgID}:services/{id}
    action: delete
```
___

# Enterprise Customization Example:

### Scenario:
ACME Bank is the sample organization. ACME bank consists of teams listed in the table below. Each of these teams interacts with their Konnect organization as per the description in the following table.

| Team | Description |
| :--- | :---------- |
| Retail | The members of the Retail Konnect team are actively building Banking applications designed for the Retail business unit of the Bank that is concerned with the Bank's customer's day-to-day banking needs. These are applications such as the Bank's mobile application for consumers. |
| Investment | The members of the Investment Konnect team are actively building Investment applications designed for providing trading services to the Bank's consumers. |
| Operations | The members of the Operations team are responsible for deploying the services built by the Retail and Investment teams to the Bank's official Production environment. From a Konnect perspective, deployment to Production environment means that the Service is proxied on the appropriate set of Gateway data-planes (which are provisioned and managed by the Operations team) and available for usage by its clients. 

The table below describes the scope of control each team has access to with respect to the Services and Runtime Groups from the Konnect perspective.

| Team | Scope of Control (Services) | Scope of Control (Runtime Groups) |
|:--|:--|:--|
| Retail | Only the Retail Team should be able manage the lifecycle of the Retail services. | Only the Retail Team should have access to deploying services in the Retail Sandbox Runtime Group. |
| Investment | Only the Investment Team should be able to manage the lifecycle of the Investment services. | Only the Investment Team should have access to deploying services in the Investment Sandbox Runtime Group. |
| Operations | Operations Team should be able to view but not manage the services of both Retail and Investment teams. | Only the Operations Team should have access to deploying a service in the Production Runtime Group. |

The goal of the steps outlined in this document is to enable a user with account ownership role (that is full root access to the Konnect account) to be able to setup the organization so that ACME Banks users can manage the lifecycle of their services according to their team membership and the types of services they are responsible for. 

### 1. Creating Teams
| Team | Description |
|:--|:--|
| retail-devs | The team of retail developers responsible for developing the retail application. |
| investment-devs | The team of investment developers responsible for developing the investment application. |
| dev-ops | The team developer operations responsible for deploying services into production. |

``` YAML
Teams:
    - id: xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx
      name: retail-devs
      users:
          - retail-dev-1
          - retail-dev-2
          - retail-dev-3
    - id: xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx
      name: investment-devs
      users:
          - investment-dev-1
          - investment-dev-2
          - investment-dev-3
    - id: xxxxxxxx-team-3333-xxxx-xxxxxxxxxxxx
      name: dev-ops
      users:
          - dev-ops-1
          - dev-ops-2
          - dev-ops-3
```

### 2. Setting Service Hub Permissions
The following table describes the 4 Services that exists in the ACME organization. Each developement team is responsible for one Frontend Service and one Backend Service in their respective business unit.

| Service ID | Description |
|:--|:--|
| retail-frontend | The frontend retail UI service managed by the Retail Team. |
| retail-backend | The backend retail API service managed by the Retail Team. |
| investment-frontend | The frontend investment UI service managed by the Investment Team. |
| investment-backend | The backend investment API service managed by the Investment Team. |

Each dev teams should be able to create Services as well as manage Services they are responsible for. In addition, the dev-ops team will need to be able to read the Services of each dev team so that they can deploy the service into the production environment.

##### Permissions for `retail-devs` to manage the retail services
```YAML
TeamPermissions:
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-svcp-1111-xxxx-xxxxxxxxxxxx # services-create
    parameters:
        region: us  # Subsequent param omitted for readability
        orgID: ACME # Subsequent param omitted for readability
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-svcp-2222-xxxx-xxxxxxxxxxxx # services-read
    parameters: 
        id: retail-frontend
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-svcp-2222-xxxx-xxxxxxxxxxxx # services-read
    parameters:
        id: retail-backend
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-svcp-3333-xxxx-xxxxxxxxxxxx # services-update
    parameters: 
        id: retail-frontend
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-svcp-3333-xxxx-xxxxxxxxxxxx # services-update
    parameters:
        id: retail-backend
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-svcp-4444-xxxx-xxxxxxxxxxxx # services-delete
    parameters: 
        id: retail-frontend
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-svcp-4444-xxxx-xxxxxxxxxxxx # services-delete
    parameters:
        id: retail-backend
```
##### Permissions for `investment-devs` to manage the investment services
```YAML
TeamPermissions:
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-svcp-1111-xxxx-xxxxxxxxxxxx # services-create
    parameters: 
        region: us  # Subsequent param omitted for readability
        orgID: ACME # Subsequent param omitted for readability
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-svcp-2222-xxxx-xxxxxxxxxxxx # services-read
    parameters:
        id: investment-frontend
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-svcp-2222-xxxx-xxxxxxxxxxxx # services-read
    parameters:
        id: investment-backend
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-svcp-3333-xxxx-xxxxxxxxxxxx # services-update
    parameters:
        id: investment-frontend
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-svcp-3333-xxxx-xxxxxxxxxxxx # services-update
    parameters:
        id: investment-backend
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-svcp-4444-xxxx-xxxxxxxxxxxx # services-delete
    parameters:
        id: investment-frontend
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-svcp-4444-xxxx-xxxxxxxxxxxx # services-delete
    parameters:
        id: investment-backend
```
##### Permissions for `dev-ops`  to retrieve all the services
```YAML
TeamPermissions:
  - id: 
    team_id:       xxxxxxxx-team-3333-xxxx-xxxxxxxxxxxx # dev-ops
    permission_id: xxxxxxxx-svcp-2222-xxxx-xxxxxxxxxxxx # services-read
    parameters:
        region: us
        orgID: ACME
        id: retail-frontend
  - id: 
    team_id:       xxxxxxxx-team-3333-xxxx-xxxxxxxxxxxx # dev-ops
    permission_id: xxxxxxxx-svcp-2222-xxxx-xxxxxxxxxxxx # services-read
    parameters:
        region: us 
        orgID: ACME
        id: investment-frontend
```

### 3. Setting Runtime Manager Permissions
The following table describes the 3 Runtime Groups that exists in the ACME organization. Each developement team is responsible for one Sandbox Runtime Group while the dev-ops team is responsible for the Production Runtime Group.

| Runtime Group | Description |
|:--|:--|
| retail-sandbox-rg | The group of retail runtimes used for development and testing by the Retail Team. |
| investment-sandbox-rg | The group of investment runtimes used for development and testing by the Investment Team. |
| production-rg | The group of production runtimes used to expose ACME services to its customers. |

##### Permissions for `retail-devs` to manage the Retail Sandbox.
```YAML
TeamPermissions:
  - id: 
    team_id:       xxxxxxxx-team-1111-xxxx-xxxxxxxxxxxx # retail-devs
    permission_id: xxxxxxxx-rtgp-3333-xxxx-xxxxxxxxxxxx # runtime-group-update
    parameters:
        region: us
        orgID: ACME
        id: retail-sandbox-rg
```
##### Permissions for `investment-devs` to manage Investment Sandbox.
```YAML
TeamPermissions:
  - id: 
    team_id:       xxxxxxxx-team-2222-xxxx-xxxxxxxxxxxx # investment-devs
    permission_id: xxxxxxxx-rtgp-3333-xxxx-xxxxxxxxxxxx # runtime-group-update
    parameters:
        region: us
        orgID: ACME
        id: investment-sandbox-rg
```
##### Permissions for `dev-ops`  to deploy services to production.
```YAML
TeamPermissions:
  - id: 
    team_id:       xxxxxxxx-team-3333-xxxx-xxxxxxxxxxxx # dev-ops
    permission_id: xxxxxxxx-rtgp-3333-xxxx-xxxxxxxxxxxx # runtime-group-update
    parameters:
        region: us
        orgID: ACME
        id: production-rg
```
