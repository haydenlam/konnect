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

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: retail-devs
spec:
  users:
  permissions:
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: investment-devs
spec:
  users:
  permissions:
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: dev-ops
spec:
  users:
  permissions:
```

### 2. Setting Service Hub Permissions
| Service | Description |
|:--|:--|
| retail-frontend | The frontend retail UI service managed by the Retail Team. |
| retail-backend | The backend retail API service managed by the Retail Team. |
| investment-frontend | The frontend investment UI service managed by the Investment Team. |
| investment-backend | The backend investment API service managed by the Investment Team. |

##### Permissions for `retail-devs` to manage the retail services
```
# Add a new service
- krn:reg/us:org/acme-bank:services:service#create
# Retrieve an existing service
- krn:reg/us:org/acme-bank:services/retail-frontend:service/*#read
- krn:reg/us:org/acme-bank:services/retail-backend:service/*#read
# Update an existing service
- krn:reg/us:org/acme-bank:services/retail-frontend:service/*#update
- krn:reg/us:org/acme-bank:services/retail-backend:service/*#update
# Remove an existing service
- krn:reg/us:org/acme-bank:services/retail-frontend:service/*#delete
- krn:reg/us:org/acme-bank:services/retail-backend:service/*#delete
```
##### Permissions for `investment-devs` to manage the investment services
```
# Add a new service
- krn:reg/us:org/acme-bank:services:service#create
# Retrieve an existing service
- krn:reg/us:org/acme-bank:services/investment-frontend:service/*#read
- krn:reg/us:org/acme-bank:services/investment-backend:service/*#read
# Update an existing service
- krn:reg/us:org/acme-bank:services/investment-frontend:service/*#update
- krn:reg/us:org/acme-bank:services/investment-backend:service/*#update
# Remove an existing service
- krn:reg/us:org/acme-bank:services/investment-frontend:service/*#delete
- krn:reg/us:org/acme-bank:services/investment-backend:service/*#delete
```
##### Permissions for `dev-ops`  to retrieve all the services
```
# Retrieve all existing service
- krn:reg/us:org/acme-bank:services/*#read
```

### 3. Setting Runtime Manager Permissions
| Runtime Group | Description |
|:--|:--|
| retail-sandbox-rg | The group of retail runtimes used for development and testing by the Retail Team. |
| investment-sandbox-rg | The group of investment runtimes used for development and testing by the Investment Team. |
| production-rg | The group of production runtimes used to expose ACME services to its customers. |

##### Permissions for `retail-devs` to manage the Retail Sandbox.
```
# Add a new runtime
- krn:reg/us:org/org1:runtime-group/retail-sandbox-rg:runtime!create
# Retrieve existing runtimes
- krn:reg/us:org/org1:runtime-group/retail-sandbox-rg:runtime/*!read
# Update existing runtimes
- krn:reg/us:org/org1:runtime-group/retail-sandbox-rg:runtime/*!update
# Delete existing runtimes
- krn:reg/us:org/org1:runtime-group/retail-sandbox-rg:runtime/*!delete
```
##### Permissions for `investment-devs` to manage Investment Sandbox.
```
# Add a new runtime
- krn:reg/us:org/org1:runtime-group/investment-sandbox-rg:runtime!create
# Retrieve existing runtimes
- krn:reg/us:org/org1:runtime-group/investment-sandbox-rg:runtime/*!read
# Update existing runtimes
- krn:reg/us:org/org1:runtime-group/investment-sandbox-rg:runtime/*!update
# Delete existing runtimes
- krn:reg/us:org/org1:runtime-group/investment-sandbox-rg:runtime/*!delete
```
##### Permissions for `dev-ops`  to deploy services to production.
```
# Add a new runtime
- krn:reg/us:org/org1:runtime-group/production-rg:runtime!create
# Retrieve existing runtimes
- krn:reg/us:org/org1:runtime-group/production-rg:runtime/*!read
# Update existing runtimes
- krn:reg/us:org/org1:runtime-group/production-rg:runtime/*!update
# Delete existing runtimes
- krn:reg/us:org/org1:runtime-group/production-rg:runtime/*!delete
```
