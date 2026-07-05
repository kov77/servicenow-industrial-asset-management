# Industrial Asset Management — ServiceNow Scoped Application

A ServiceNow application for tracking industrial equipment, maintenance requests, and spare parts inventory — built to reflect real Industrial IT/OT (Operational Technology) use cases.

## Overview

This application manages the full lifecycle of equipment maintenance in a manufacturing environment: from tracking physical assets on the CMDB, to logging maintenance requests against specific equipment, to managing spare parts inventory consumed during repairs.

The project was built as a hands-on learning exercise focused on production-grade ServiceNow architecture patterns — proper table extension, reference-based relationships, many-to-many data modeling, and server-side scripting with GlideRecord.

## Data Model

Three interconnected tables, each following ServiceNow best practices for the type of data they represent:

| Table | Extends | Purpose |
|---|---|---|
| **Equipment** | `cmdb_ci` | Physical assets tracked as Configuration Items — conveyors, press machines, compressors, CNC machines, robot arms |
| **Maintenance Request** | `task` | Repair/maintenance tickets, inheriting standard ITSM fields (number, priority, state, assignment) plus custom fields for equipment reference and downtime tracking |
| **Spare Part** | *(standalone)* | Inventory of parts used in repairs, with stock quantity and compatibility tracking |
| **Parts Used** | *(junction table)* | Many-to-many relationship linking Maintenance Requests to the Spare Parts consumed in each repair, with quantity tracking |

### Why extend `cmdb_ci` and `task`?

- **Equipment extends `cmdb_ci`** — this places equipment directly in the CMDB, ServiceNow's central asset registry, instead of maintaining a disconnected custom table. This enables future integration with Service Mapping, Impact Analysis, and other CMDB-aware modules.
- **Maintenance Request extends `task`** — this is the same base table used by Incident, Problem, and Change Management. It provides built-in SLA tracking infrastructure, assignment rules, and standard ACL patterns for free, rather than rebuilding them from scratch.

## Key Technical Features

- **Reference fields** connecting Maintenance Request → Equipment (instead of free-text fields), enabling reliable reporting and preventing data duplication
- **Many-to-many relationship** between Maintenance Request and Spare Part via a junction table, with a related list surfaced directly on the Maintenance Request form
- **Server-side GlideRecord scripting** — a Business Rule that cross-updates the Spare Part table when parts are consumed, automatically decrementing stock quantity
- **Flow Designer automations** for priority-based escalation and status-driven record updates
- **UI Policies** for conditional field requirements based on priority and status
- **Role-based access control** — `asset_technician` and `asset_manager` roles with distinct CRUD permissions per table

## Sample Business Rule (GlideRecord cross-table update)

```javascript
// Deduct spare part quantity when a part is logged against a maintenance request
(function executeRule(current, previous /*null when async*/) {
    var part = new GlideRecord('x_2103100_indust_0_spare_part');
    if (part.get(current.spare_part)) {
        part.quantity_in_stock = part.quantity_in_stock - current.quantity_used;
        part.update();
    }
})(current, previous);
```

## Roles

| Role | Equipment | Maintenance Request | Spare Part |
|---|---|---|---|
| `asset_technician` | Read, Write | Create, Read, Write | Read only |
| `asset_manager` | Full access | Full access | Full access |

## Tech Stack

- ServiceNow (Australia release), Scoped Application
- App Engine Studio for data modeling
- Flow Designer for process automation
- GlideRecord / server-side JavaScript for business logic
- Git-based source control integration

## Status

Actively developed — this repository reflects an incremental commit history as features are added, rather than a single bulk import.
