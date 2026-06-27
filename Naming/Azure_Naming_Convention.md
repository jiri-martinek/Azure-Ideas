# Azure Resource Naming Convention

**Hierarchical, Context-Driven Naming Standard**
*Standard & Implementation Guide*

Version 1.0 — June 2026

## Table of Contents

- [1. Introduction & Scope](#1-introduction--scope)
  - [1.1 Why This Convention](#11-why-this-convention)
- [2. Core Naming Principles](#2-core-naming-principles)
- [3. Standard Name Structure](#3-standard-name-structure)
  - [3.1 Logical Components](#31-logical-components)
  - [3.2 The Inheritance Principle](#32-the-inheritance-principle)
  - [3.3 Mapping Examples](#33-mapping-examples)
- [4. Separators](#4-separators)
- [5. Naming by Scope](#5-naming-by-scope)
  - [5.1 Management Groups](#51-management-groups)
  - [5.2 Subscriptions](#52-subscriptions)
  - [5.3 Resource Groups](#53-resource-groups)
  - [5.4 Resources](#54-resources)
- [6. Naming by Resource Category](#6-naming-by-resource-category)
  - [6.1 Compute & Web](#61-compute--web)
  - [6.2 Networking](#62-networking)
  - [6.3 Databases](#63-databases)
  - [6.4 Storage](#64-storage)
  - [6.5 Identity & Security](#65-identity--security)
  - [6.6 Monitoring, Integration & Analytics](#66-monitoring-integration--analytics)
  - [6.7 Resource Constraints Reference](#67-resource-constraints-reference)
- [7. Entra ID & Service Account Naming](#7-entra-id--service-account-naming)
  - [7.1 Service Accounts](#71-service-accounts)
  - [7.2 Entra ID Objects](#72-entra-id-objects)
- [8. Special Naming Rules](#8-special-naming-rules)
  - [8.1 Resource Types with Hard Limits](#81-resource-types-with-hard-limits)
  - [8.2 Storage Account Decomposition](#82-storage-account-decomposition)
  - [8.3 Global Uniqueness Qualifier](#83-global-uniqueness-qualifier)
  - [8.4 Network Names vs Azure Resource Names](#84-network-names-vs-azure-resource-names)
  - [8.5 Region in the Name (Exception)](#85-region-in-the-name-exception)
  - [8.6 Auto-Generated Names](#86-auto-generated-names)
  - [8.7 Case Handling](#87-case-handling)
- [9. Practical Deployment Examples](#9-practical-deployment-examples)
  - [9.1 Hub-Spoke Architecture](#91-hub-spoke-architecture)
  - [9.2 New Application Landing Zone](#92-new-application-landing-zone)
- [10. Naming & Tagging Alignment](#10-naming--tagging-alignment)
- [11. Governance, Validation & Compliance](#11-governance-validation--compliance)
  - [11.1 Governance Principles](#111-governance-principles)
  - [11.2 Abbreviation Registry](#112-abbreviation-registry)
  - [11.3 Exception Management](#113-exception-management)
  - [11.4 Naming Enforcement: Generate vs Verify](#114-naming-enforcement-generate-vs-verify)

# 1. Introduction & Scope

This document defines the naming convention for Azure resources, resource groups, and subscriptions. It establishes a single, consistent standard that all teams and automation systems should follow when creating or modifying Azure resources.

**The Cloud Adoption Framework (CAF) recommends a principle rather than a fixed scheme: **names should follow a consistent format that carries the information needed to identify a resource. It offers an *illustrative example* (such as pip-sharepoint-prod-westus-001). CAF explicitly expects each organization to define its own convention on top of that principle. This document is that convention.

**It is built for clarity and scale. **Two design choices give it its main advantages. First, it is **hierarchical and context-driven**: a resource name inherits its prefix from the resource group, and the resource group inherits its prefix from the subscription — so names sort and group naturally by application, and the hierarchy itself tells you where a resource belongs. Second, it follows an **“only what is needed” principle**: information that Azure already carries elsewhere — most notably the resource type and the region — stays in tags and properties rather than being repeated in the name. The result is a name that is shorter, easier to read, and far less likely to collide with Azure's length limits, while losing none of the information, which remains available from Azure and from tags.

The convention applies to all Azure resources across all subscriptions, management groups, and environments. It covers infrastructure (compute, networking, storage), platform services (identity, security, monitoring), data services, and supporting DevOps systems.

This document draws on Microsoft's Cloud Adoption Framework where relevant, specifically:

- Recommendations on naming structure: learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming

- Scope and limitations of the name format: learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules

- Reserved expressions: learn.microsoft.com/azure/azure-resource-manager/templates/error-reserved-resource-name

- Resource type abbreviations: learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations

## 1.1 Why This Convention

**The advantages of this convention are concrete:**

| **What the convention does** | **The benefit it gives you** |
| --- | --- |
| Inherits each name's prefix from its parent scope | Resources sort and group by application automatically in the portal; the hierarchy is visible in the name |
| Keeps the resource type out of the name | Shorter names; “list all VMs” is a native filter, not a text-parsing exercise; no drift when a type is renamed |
| Keeps the region out of the name | Shorter names; region comes from the resource's location and a tag, with no risk of a stale region string |
| Carries only the components a scope hasn't already fixed | Far fewer characters used, so storage / Key Vault / hostname limits are met without awkward shortening |
| Uses minimal, purposeful separators | Readable names that still parse cleanly, and that fit constrained types via the x fallback |
| Treats auto-generated names as accepted exceptions | No fighting Azure over names you don't control; governance falls back to tags |

**This approach is fully within the spirit of CAF. **Microsoft's guidance is advisory — names *should* follow a consistent format and *ideally* include what is needed to identify a resource — and CAF presents its own pattern as an example, leaving the choice of scheme to each organization. This convention takes up that invitation with a scheme tuned for readability and scale. Supporting references: the CAF naming guidance (…/resource-naming), the per-type rules and limits (…/resource-name-rules), and Microsoft's CAF PSRule definitions, which phrase the conventions as recommendations.

# 2. Core Naming Principles

The following principles govern every naming decision. They must be understood before applying the detailed rules in later sections.

- **Hierarchical inheritance — **each scope level inherits the name (prefix) of its parent. A resource group name begins with its subscription name; a resource name begins with its resource group name. Consistency of this chain is the backbone of the convention.

- **Context over duplication — **a name carries only what cannot be cheaply derived from the resource itself (its type and region come from Azure, not the name). What the name does carry, it carries in full: each name restates the components inherited from its parent scope as its prefix, so the name is complete on its own and the hierarchy simply agrees with it. This is inheritance, not omission — the prefix is present in every name, not left to be looked up.

- **No resource type in the name — **Azure exposes the resource type as a native property (a column in the portal, a field in Resource Graph). Listing “all VMs” is a built-in filter, not a name-parsing exercise. The type is therefore omitted from the name.

- **No region in the name — **the region is a property of the resource and belongs in a tag or is read from the resource location. It is omitted from the name except in the rare cases described in Section 8.

- **Omit unused parts — **name components that do not apply to a given resource are omitted, but the relative order of the remaining components is always preserved.

- **Minimal, purposeful separators — **separators cost characters but add readability. Use as few as possible, and only where they carry meaning or are required to parse the name.

- **Hyphen primary, “x” fallback — **the hyphen (-) is the standard separator. Where hyphens are not allowed (e.g. storage accounts), the letter “x” is used as a visual delimiter.

- **Auto-generated names are accepted exceptions — **resources named automatically by Azure or by deployment tools are exempt from naming compliance and are governed through tagging instead.

# 3. Standard Name Structure

## 3.1 Logical Components

A name is assembled from the following logical components, in this fixed left-to-right order:

| **Component** | **Meaning** | **Example** |
| --- | --- | --- |
| application | Application name | myapp |
| application_instance | Instance / deployment of the application | second |
| environment | Environment, single letter: d (dev), t (test), p (prod) | t |
| application_layer | Application layer or infrastructure unit | sql, web |
| resource_name | Name or functional role of the resource | node |
| resource_instance | Sequence number of the resource instance | 01, 02 |
| subresource_name | Name of a dependent sub-resource (optional) | N (nic), D (disk) |
| subresource_instance | Sequence number of the sub-resource instance | 01, 02 |

**Note: **compared with the generic CAF pattern, there is no resource_type and no region component. The type is carried by Azure itself; the region lives in tags and in the resource's own location property.

**On the layer component: **the application_layer should be drawn from a short, standardized set of abbreviations (e.g. fe, be, db, dt, net, sec, mon), agreed once and kept in the registry (Section 11.2), because the layer appears in almost every name. Note that a layer is *not* a resource type: one layer normally holds several different Azure types (the be layer contains VMs, disks, and NICs alike). The layer says which part of the application a resource belongs to; the type is read from Azure.

**Fallback when there is no distinguishing role: **occasionally a resource has no role that distinguishes it — there is a single resource of its type, and its identity is already fully determined by the higher level (the resource group / scope). In that case resource_name has nothing meaningful to carry, and it falls back to the resource type itself. This is not a violation of the “no type in the name” rule: the type is not added as extra metadata, it is used as the last available role name once the hierarchy is exhausted.

**When this fallback applies, use the official CAF resource type abbreviation **(e.g. vm, st, kv) rather than an ad-hoc form. If no official abbreviation exists for the type, use a repeatable custom one and register it (see Section 11.2). The CAF abbreviation list is at learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations.

## 3.2 The Inheritance Principle

This is the heart of the convention. The logical components are mapped onto the physical hierarchy subscription → resource group → resource. The mapping is not fixed — it depends on the size and structure of the deployment.

- **Subscription and resource group consume a contiguous prefix of the components **(in the order from Section 3.1), from the left.

- **The resource name begins with the inherited prefix and appends the remaining components **— it is the full chain (prefix + its own role and instance), not just the leftover part. The resource name therefore contains everything above it, spelled out.

- **Each level restates the components above it as its prefix **— the hierarchy and the name carry the same leading components, so they always agree. Inheritance means the prefix is part of every lower name, not that it is dropped.

- **Only a contiguous left-anchored prefix may be consumed by the scope **— never with a gap (e.g. application may not be taken while skipping instance). This keeps the derivation unambiguous.

**Every resource name is self-describing and unique on its own. **A resource name is not a bare suffix — it carries the full inherited prefix, so the complete name (e.g. myappsecondt-sql-node01) already states the application (myapp), instance (second), environment (t), layer (sql) and the specific resource (node01) in one string. Reading the name alone, without consulting its subscription or resource group, is enough to know exactly what it is. There is no “look it up elsewhere” step.

**Context is reinforcing, not required. **The subscription and resource group repeat the same leading components, so the position in the hierarchy and the name agree with each other — but the name does not depend on that position to be understood. This is the opposite of a flat scheme where a name like node01 would indeed be meaningless without context; here the inherited prefix is part of the resource name itself, by construction. (The only thing that is genuinely context-bound is an artificially truncated fragment, which never occurs because the convention always emits the full name.)

## 3.3 Mapping Examples

The same convention produces different name distributions depending on how much of the hierarchy each scope absorbs. The three examples below are illustrative, not an exhaustive catalogue.

### Variant A — Small deployment

| **Scope** | **Adds these components** | **Resulting full name** |
| --- | --- | --- |
| subscription | application | myapp-sub |
| resource group | instance + environment | myappsecondt-rg |
| resource | layer + resource + instance | myappsecondt-sql-node01 |

### Variant B — Large deployment

| **Scope** | **Adds these components** | **Resulting full name** |
| --- | --- | --- |
| subscription | application + instance + environment | myappsecondt-sub |
| resource group | layer | myappsecondt-sql-rg |
| resource | resource + instance | myappsecondt-sql-node01 |

### Variant C — Medium deployment

| **Scope** | **Adds these components** | **Resulting full name** |
| --- | --- | --- |
| subscription | application + instance | myappsecond-sub |
| resource group | environment + layer | myappsecondt-sql-rg |
| resource | resource + instance | myappsecondt-sql-node01 |

*In all variants the same rule holds: the RG name begins with (the prefix of) its subscription, and the resource name begins with (the prefix of) its RG. This chained inheritance is what the optional policy in Section 11 validates.*

**Note how to read the table: **the middle column lists only the components each scope *adds*; the resulting name still contains everything inherited from above. In Variant B the resource scope adds just resource + instance, yet the full resource name is myappsecondt-sql-node01 — the inherited myappsecondt-sql is part of the resource's own name, not something kept only in the resource group. The name is read as a whole; nothing has to be looked up to interpret it.

# 4. Separators

Separators improve readability but consume characters from tight length limits. The rule is: as few as possible, and only where they carry meaning or are needed to parse a name.

- **Between scope levels (sub / rg / resource): **no dedicated separator — the boundary is carried by Azure's structure itself, since these are three separate entities / fields.

- **environment (single letter): **appended without a separator — a fixed length of one character is enough to read it unambiguously.

- **Between variable-length components within one name **(e.g. layer vs resource_name): a separator is used — otherwise they cannot be reliably told apart.

**Layer is the only component bounded by a separator on both sides **(e.g. myappsecondt-sql-node01, where sql sits between two hyphens). Every other component either glues onto its neighbour (instance onto environment, resource_name onto its instance) or sits at the boundary of the name or scope. Being delimited on both sides, the layer is the one component a policy can isolate cleanly and validate on its own against an allowed value set (e.g. sql | web | …).

The primary separator is - (hyphen). Where the hyphen is not allowed (see storage), x is used as the visual delimiter. Because the resource name inherits the full RG name as a prefix, in practice there is exactly one separator between the inherited prefix and the new content, e.g. myappsecondt-sql-node01.

**Validation granularity trade-off: **gluing components together without a separator (e.g. myappsecond) means a policy cannot validate them individually — only that the name starts with the correct inherited prefix. If individual components must be validated against an allowed set of values, those components need an explicit separator. Choose based on how granular the validation needs to be; the rest is readability.

# 5. Naming by Scope

Azure organizes resources in a hierarchy: Tenant → Management Groups → Subscriptions → Resource Groups → Resources. Each level has its own naming form within the overall convention.

## 5.1 Management Groups

Management groups provide governance scope above subscriptions. Their names are short and descriptive, reflecting an organizational or functional grouping. Pattern: mg-<unit>.

| **Example** | **Purpose** |
| --- | --- |
| mg-myappgroup | Top-level group for the Myapp organization |
| mg-platform | Shared platform / hub services |
| mg-apps | Application landing zones |
| mg-sandbox | Isolated sandbox subscriptions |

## 5.2 Subscriptions

Subscriptions are the primary boundary for billing, access control, and limits. A subscription name ends with -sub and carries the contiguous prefix of components assigned to it by the chosen mapping (Section 3.3). Pattern: <scope_prefix>-sub.

| **Example** | **Mapping (components absorbed by the subscription)** |
| --- | --- |
| myapp-sub | application (Variant A) |
| myappsecond-sub | application + instance (Variant C) |
| myappsecondt-sub | application + instance + environment (Variant B) |

## 5.3 Resource Groups

Resource groups are logical containers for resources that share a lifecycle, access control, or deployment unit. An RG name begins with its subscription prefix, may add the next contiguous components (e.g. layer), and ends with -rg. Pattern: <subscription_prefix>[<sep><next components>]-rg.

| **Example** | **Contents** |
| --- | --- |
| myappsecondt-rg | All resources for the MyappSecond test deployment (Variant A) |
| myappsecondt-sql-rg | SQL layer of the MyappSecond test deployment (Variants B/C) |
| myappsecondt-web-rg | Web layer of the same deployment |

## 5.4 Resources

A resource name begins with its resource group prefix and adds the remaining contiguous components (resource name and instance, optionally a dependent sub-resource). Pattern: <rg_prefix><sep><resource_name><resource_instance>[<sep><subresource_name><subresource_instance>].

| **Example** | **Meaning** |
| --- | --- |
| myappsecondt-sql-node01 | First SQL node in the MyappSecond test deployment |
| myappsecondt-sql-node01 | Same, with the layer carried at resource level (Variant B) |
| myappsecondt-sql-node01-n01 | First dependent NIC of that node |

# 6. Naming by Resource Category

**These tables are illustrative, not a set of independent rules. **The binding specification is the grammar and inheritance logic in Sections 3 and 4. Each “illustrative form” below is simply that grammar *applied* to a common resource type under one particular scope mapping (Variant B). Do not treat a form such as <app>-<layer>-node<nn> as a mandatory template: the components it shows, and where they sit, depend on the mapping you chose in Section 3.3. A different mapping legitimately produces a different distribution of components across the name. Read these as worked examples, not per-type definitions to copy verbatim.

**Reading the forms: **<app> is the subscription prefix (application + instance, plus environment where used), <layer> is the application layer carried by the resource group, and the tail is the resource's own role and instance. A full name is therefore <app>-<layer>-<resource><nn>. The resource type is shown only as context — it is never part of the name. Examples use a short instance (usa1) and the Variant B mapping; the prefix myappusa1t means application myapp, instance usa1, environment t.

**Length is the hard constraint. **Examples are kept short on purpose so they fit Azure's limits (storage / Key Vault at 24 characters are the tightest). This is also why **layer names should be standardized as short 2–3 letter abbreviations** in the registry (Section 11.2), for example: fe (frontend / web), be (backend / compute), db (database), dt (data: storage, lakes), sec (security / identity), net (network), mon (monitoring). These are examples, not a fixed list — standardize the set that fits your estate.

**Environment is optional. **Like any component, it can be omitted as long as the name still splits cleanly into subscription / resource group / resource. In particular it is dropped where only production makes sense — for example a subscription for Microsoft Fabric capacity or for Sentinel, where there is no dev/test counterpart.

## 6.1 Compute & Web

Compute resources include virtual machines, scale sets, availability sets, disks, and web/app services. Note that the VM name also becomes the initial OS hostname (Windows: 15 chars, Linux: 64 chars), which may force shortening — see Section 8.

| **Azure resource (type, not in name)** | **Illustrative form** | **Example** |
| --- | --- | --- |
| Virtual machine | <app>-be-node<nn> | myappusa1t-be-node01 |
| VM scale set | <app>-fe-<role><nn> | myappusa1t-fe-front01 |
| Availability set | <app>-be-<role> | myappusa1t-be-avail |
| Managed disk (data) | <vm>-d<nn> | myappusa1t-be-node01-d01 |
| App Service plan | <app>-fe-<role> | myappusa1t-fe-plan |
| Web app (global) | <app>-fe-<role> | myappusa1t-fe-portal |

## 6.2 Networking

Networking resources define the security and connectivity boundaries of the platform; consistent naming is therefore particularly important. Dependent resources (NIC, public IP) attach to their parent resource by name.

| **Azure resource (type, not in name)** | **Illustrative form** | **Example** |
| --- | --- | --- |
| Virtual network | <app>-net-net<nn> | myappusa1t-net-net01 |
| Subnet | <app>-net-net<nn>-<role> | myappusa1t-net-net01-app |
| Network interface (NIC) | <vm>-n<nn> | myappusa1t-be-node01-n01 |
| Public IP | <parent>-pip<nn> | myappusa1t-fe-portal-pip01 |
| Network security group | <app>-net-<role> | myappusa1t-net-allow |
| Route table | <app>-net-rt<nn> | myappusa1t-net-rt01 |
| VNet peering | <app>-net-peer-<dst> | myappusa1t-net-peer-hub |

**Reserved subnet names **required by Azure are kept verbatim: GatewaySubnet, AzureFirewallSubnet, AzureBastionSubnet. Peering names contain both the source and the destination network identification.

**Peering is bidirectional: **a peering myappusa1t-net-peer-hub on the spoke side must have a corresponding peering hub-net-peer-myappusa1t configured on the hub side. Both directions must exist for connectivity to work.

## 6.3 Databases

Many database resources have globally unique naming requirements because their names become part of the connection endpoint. For globally-scoped resources, consider whether a company qualifier is needed to avoid collisions (Section 8).

| **Azure resource (type, not in name)** | **Illustrative form** | **Example** |
| --- | --- | --- |
| Azure SQL server (global) | <app>-db-sql<nn> | myappusa1t-db-sql01 |
| Azure SQL database | <server>-<dbname> | myappusa1t-db-sql01-orders |
| Cosmos DB (global) | <app>-db-cos<nn> | myappusa1t-db-cos01 |
| Azure Cache for Redis (global) | <app>-db-rds<nn> | myappusa1t-db-rds01 |

## 6.4 Storage

Storage accounts are the most constrained resource type: 3–24 characters, lowercase letters and numbers only, **no hyphens**, and globally unique worldwide. The standard hyphen-separated form cannot be used — the x delimiter and an all-lowercase form apply (Section 8). The short layer abbreviation matters most here.

| **Azure resource (type, not in name)** | **Illustrative form** | **Example** |
| --- | --- | --- |
| Storage account (global) | <app>x<layer>x<purpose> | myappusa1txdtxlake |
| Container registry (global) | <app>x<layer>x<purpose> | myappusa1txintxacr |
| Blob container | <purpose>-<qualifier> | backups-daily |

**On the storage example: **myappusa1txdtxlake = application myapp, instance usa1, environment t, data layer dt, purpose lake (a data lake). At 18 characters it sits comfortably under the 24-character limit; dropping the environment where only production exists frees three more.

## 6.5 Identity & Security

Identity resources appear in RBAC assignments, audit logs, and access policies, so they must be named with care. Service-account naming for Entra ID objects is covered in Section 7.

| **Azure resource (type, not in name)** | **Illustrative form** | **Example** |
| --- | --- | --- |
| User-assigned managed identity | <app>-sec-<role> | myappusa1t-sec-deploy |
| Key Vault (global, 24 chars) | <app>-sec-kv<nn> | myappusa1t-sec-kv01 |
| Bastion host | <app>-net-bas | myappusa1t-net-bas |

## 6.6 Monitoring, Integration & Analytics

| **Azure resource (type, not in name)** | **Illustrative form** | **Example** |
| --- | --- | --- |
| Log Analytics workspace | <app>-mon-log<nn> | myappusa1t-mon-log01 |
| Application Insights | <app>-mon-appi | myappusa1t-mon-appi |
| Data Factory (global) | <app>-int-adf<nn> | myappusa1t-int-adf01 |
| Service Bus namespace (global) | <app>-int-sb<nn> | myappusa1t-int-sb01 |
| Event Hub | <namespace>-<name> | myappusa1t-int-sb01-evt |

## 6.7 Resource Constraints Reference

This table is a curated quick reference of the most common Azure resource types and their **naming constraints**. Unlike a CAF prefix catalogue, it does not list prefixes — this convention does not put the type in the name. Instead it lists the three things that actually drive naming decisions here: the **scope** (which determines whether the name must be globally unique — see Section 8.3), the **length limit** (which determines whether shortening is needed — see Section 8.1), and the **allowed characters** (which determines whether the x fallback separator is required).

It is a representative subset, not exhaustive. Constraints reflect current Azure rules at the time of writing and may change; for unlisted or unfamiliar types, verify against the official Microsoft resource naming rules. Types marked **Global** must be unique worldwide; types with **no hyphens** require the lowercase + x form.

| **Resource** | **Scope** | **Length** | **Characters / notes** |
| --- | --- | --- | --- |
| Virtual machine | Resource group | 1-15 Win / 1-64 Lin | Alphanumeric + hyphen; Win no period/end-hyphen |
| VM scale set | Resource group | 1-15 Win / 1-64 Lin | Same host rules as VM |
| Availability set | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Managed disk | Resource group | 1-80 | Alphanumeric, underscore, hyphen |
| App Service plan | Resource group | 1-40 | Alphanumeric, hyphen |
| Web app / Function app | Global (DNS) | 2-60 | Alphanumeric, hyphen; no start/end hyphen |
| App Service environment | Resource group | 1-36 | Alphanumeric, hyphen |
| Virtual network | Resource group | 2-64 | Alphanumeric, underscore, period, hyphen |
| Subnet | Virtual network | 1-80 | Alphanumeric, underscore, period, hyphen |
| Network interface (NIC) | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Public IP | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Network security group | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Route table | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Load balancer | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Application gateway | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Azure Firewall | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Private endpoint | Resource group | 2-64 | Alphanumeric, underscore, hyphen |
| Private DNS zone | Resource group | 1-63 | Per-label alphanumeric, underscore, hyphen |
| VPN / virtual net gateway | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Bastion host | Resource group | 1-80 | Alphanumeric, underscore, period, hyphen |
| Traffic Manager profile | Global | 1-63 | Alphanumeric, hyphen, period |
| Front Door (classic) | Global | 5-64 | Alphanumeric, hyphen |
| Storage account | Global | 3-24 | LOWERCASE alphanumeric ONLY — no hyphens |
| Container registry | Global | 5-50 | Alphanumeric ONLY — no hyphens |
| Blob container | Storage account | 3-63 | Lowercase, number, hyphen; no consecutive hyphen |
| File share | Storage account | 3-63 | Lowercase, number, hyphen |
| Storage Sync Service | Resource group | 1-260 | Alphanumeric, space, period, hyphen, underscore |
| Key Vault | Global | 3-24 | Alphanumeric, hyphen; no consecutive hyphen |
| Managed identity (user) | Resource group | 3-128 | Alphanumeric, hyphen, underscore |
| Recovery Services vault | Resource group | 2-50 | Alphanumeric, hyphen; start with letter |
| Backup vault | Resource group | 2-50 | Alphanumeric, hyphen |
| Azure SQL server | Global | 1-63 | LOWERCASE, number, hyphen; no start/end hyphen |
| Azure SQL database | SQL server | 1-128 | Most chars except <>*%&:\/? |
| SQL Managed Instance | Global | 1-63 | Lowercase, number, hyphen |
| Cosmos DB account | Global | 3-44 | Lowercase, number, hyphen |
| Azure Cache for Redis | Global | 1-63 | Alphanumeric, hyphen; no consecutive hyphen |
| PostgreSQL / MySQL | Global | 3-63 | Lowercase, number, hyphen; no start/end hyphen |
| Data Factory | Global | 3-63 | Alphanumeric, hyphen |
| Databricks workspace | Resource group | 3-64 | Alphanumeric, underscore, hyphen |
| Synapse workspace | Global | 1-50 | Lowercase, number, hyphen |
| Event Hub namespace | Global | 6-50 | Alphanumeric, hyphen; start with letter |
| Service Bus namespace | Global | 6-50 | Alphanumeric, hyphen; start with letter |
| API Management | Global | 1-50 | Alphanumeric, hyphen |
| Cognitive Services | Resource group | 2-64 | Alphanumeric, hyphen |
| Machine Learning workspace | Resource group | 3-33 | Alphanumeric, hyphen |
| Log Analytics workspace | Resource group | 4-63 | Alphanumeric, hyphen |
| Application Insights | Resource group | 1-260 | Most chars except <>*%&:\?+/ |
| AKS cluster | Resource group | 1-63 | Alphanumeric, underscore, hyphen |
| Container app | Resource group | 2-32 | LOWERCASE, number, hyphen |
| Resource group | Subscription | 1-90 | Alphanumeric, underscore, period, paren, hyphen |
| Management group | Tenant | 1-90 | Alphanumeric, underscore, period, paren, hyphen |

**Full, authoritative lists (live sources): **this table is a curated extract. For the complete and current rules, always consult the Microsoft sources directly, as they are updated over time:

- Naming rules & restrictions (length, characters, scope) per resource type: learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules

- CAF resource type abbreviations: learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations

# 7. Entra ID & Service Account Naming

Entra ID directory objects are not Azure resources, but consistent naming supports discoverability, automation, and audit. Objects synchronized from on-premises Active Directory keep their on-premises names; the conventions below apply to cloud-native objects.

## 7.1 Service Accounts

Service accounts use a constant prefix that identifies the account type, followed by application, instance/environment, and a service specification. Keep names as short as possible, since they may also act as a SAM account name. Pattern: <prefix>-<app>-<instance>-<service>.

| **Prefix** | **Meaning** | **Example** |
| --- | --- | --- |
| svc- | Account used only for app-to-app authentication | svc-myapp-secondt-sql |
| sys- | Account also used to run a service | sys-myapp-secondt-agent |
| svp- | Service principal (e.g. DevOps deployment) | svp-myapp-secondt-trf1 |
| sva- | Hybrid (synced) variant of svc- | sva-myapp-secondt-sql |

**Managed identities: **system-assigned identities are named by Azure and are not subject to this convention. User-assigned managed identities follow the standard resource naming from Section 6.5.

## 7.2 Entra ID Objects

Directory objects use a short uppercase prefix indicating the object type, followed by descriptive scope and purpose components.

| **Object type** | **Pattern** | **Example** |
| --- | --- | --- |
| Security group | SG-<Type>-<Scope>-<Purpose> | SG-Access-Myapp-Sql |
| App registration | APP-<App>-<Purpose>-<Env> | APP-Myapp-OIDC-Prod |
| Service principal | SPN-<App>-<Purpose>-<Env> | SPN-Myapp-Deploy-Prod |
| Conditional Access policy | CA-<Action>-<Target>-<Cond> | CA-RequireMFA-Admins-AllLoc |

# 8. Special Naming Rules

## 8.1 Resource Types with Hard Limits

Most resources fit the full hierarchical name comfortably. A small number of types have aggressive constraints that require a dedicated strategy rather than a compromise to the whole convention.

| **Type** | **Constraint** | **Rule** |
| --- | --- | --- |
| Storage account | 3–24 chars, lowercase alphanumeric, NO hyphens, global | lowercase, x separator |
| Container registry | 5–50 chars, alphanumeric only, global | lowercase, no separators |
| Key Vault | 3–24 chars, global | shorten components, watch length |
| VM (Windows hostname) | computer name max 15 chars | in a deep scope mapping, the resource carries few tokens — usually fits |

**Length is helped by the inheritance principle itself: **the deeper the scope (the more components absorbed by subscription and RG), the shorter the resource name. For length-constrained types, prefer a deeper mapping (Variant B) so the resource carries only its tail.

## 8.2 Storage Account Decomposition

The x character is used as a visual word boundary because hyphens are forbidden and x rarely appears at component boundaries. Example decomposition of myappusa1txdtxlake:

| **Segment** | **Value** | **Meaning** |
| --- | --- | --- |
| application + instance | myappusa1 | application myapp, instance usa1 |
| environment | t | test (omit where only prod exists) |
| separator | x | visual delimiter (hyphen not allowed) |
| layer | dt | data layer (abbreviated) |
| separator | x | visual delimiter |
| purpose | lake | purpose of the account (data lake) |

**Total: 18 characters (limit is 24). **The short layer abbreviation and instance keep it well clear of the limit. A company qualifier is optional and should be added only where global uniqueness genuinely requires it — it is not a mandatory component of every storage name.

## 8.3 Global Uniqueness Qualifier

The resource_instance component resolves collisions *within* your own scope (instance 01 vs 02). It does *not* resolve a *global* collision — where a globally-scoped resource (storage account, Key Vault, SQL server, Container Registry, Cosmos DB) would clash with a name already taken by an unrelated organization anywhere in Azure. These are two different problems and need two different fields.

For global-scope resources only, and only when a real collision occurs, append a short uniqueness qualifier — a 2–3 character company identifier or random suffix. Example: myappusa1txdtxlake → myappusa1txdtxlakexy where xy is the qualifier.

- **Optional, not mandatory. **Do not add it to every global resource by default — only where uniqueness genuinely requires it. Adding it everywhere wastes characters from the 24-char limits.

- **Prefer a stable company identifier **over a random suffix where possible — it stays meaningful and deterministic for automation.

- **Document each qualifier **so the same company identifier is always spelled the same way.

## 8.4 Network Names vs Azure Resource Names

**An Azure resource name is not the same thing as a network name or OS hostname. **Azure services are represented on the network by their endpoints, and the network name (if any) is usually independent of the Azure resource name.

This matters most for VMs and VM-type appliances. When a VM is created, its initial hostname is derived from the Azure resource name (often by shortening it to fit the Windows 15-character computer-name limit). But the hostname typically follows a separate, pre-existing on-premises naming convention — extended with the new “Azure” datacenter — and will therefore differ from the Azure resource name. In practice this means the hostname must be changed after the VM is created.

**Scope note: **network and hostname naming is out of scope for this document. It is governed by the existing network/hostname convention. This document covers only the naming of Azure resources and entities.

## 8.5 Region in the Name (Exception)

The region is normally omitted. The single justified exception is when instances of the same logical resource exist in different regions because of service availability (e.g. Azure OpenAI or Microsoft Fabric capacity that is tied to one hub but provisioned in another region). In that case append a short region code so operators can tell the instances apart. The region code must reflect the actual deployment region.

**Use a standard 2–3 character region abbreviation **(e.g. we = West Europe, ne = North Europe, sdc = Sweden Central) rather than inventing one. Microsoft does not publish a single canonical short-code table, so derive codes consistently and — exactly as with type abbreviations — if you use a custom region code, register it (Section 11.2) so the same region is always written the same way. The Azure region list and geo-codes are referenced from the CAF abbreviation guidance at learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations.

## 8.6 Auto-Generated Names

Many resources are created as side effects of other deployments — NICs, OS disks, AKS node resource groups, ARM/Terraform deployment objects. These do not follow the convention, and that is an explicitly accepted exception.

- Where the tool lets you specify the name (e.g. an explicit NIC name in Terraform), provide a compliant one.

- Where the name cannot be controlled (OS disk GUID suffixes, AKS node RG), accept it as-is. Do not rename manually — it can cause service disruption.

- All auto-generated resources must still be tagged, so governance and cost tracking work even when the name does not comply.

Common auto-generated formats teams will encounter:

| **Resource** | **Auto-generated format** | **Controllable?** |
| --- | --- | --- |
| OS disk | <vm>_OsDisk_1_<guid> | No (GUID suffix) |
| NIC (with VM) | <vm><random>_z<zone> | Partly (can pre-create) |
| AKS node RG | MC_<rg>_<cluster>_<region> | No (fully Azure-controlled) |
| Deployment object | timestamped / hashed name | No |
| Storage sub-service | default | No (fixed by Azure) |

## 8.7 Case Handling

**All generated names are lowercase. **Azure treats resource and resource group names as case-insensitive (except where a type explicitly requires lowercase, such as storage accounts), which means casing carries no meaning — myappsecondt and MyappSecondT are the same resource. To avoid any ambiguity, this convention fixes a single canonical form: lowercase.

**This is not merely cosmetic. **When a name is read back through Azure's APIs, the returned value may have different casing than what was originally set, and the casing is not guaranteed to be consistent across APIs. Microsoft's own guidance is to always compare names case-insensitively. Tooling that does a case-sensitive comparison — notably Terraform — can see a name come back in unexpected case and conclude the resource must be destroyed and recreated. Standardizing on one lowercase form removes that whole class of state-drift problem.

The rules:

- **Generate lowercase. **The naming module lowercases every name as its final step (Section 11.4), for all resource types, not just storage.

- **Compare case-insensitively. **Any policy, script, or pipeline that matches on a name must do so case-insensitively, and must never assume Azure echoes a name back in the case it was set.

- **Storage-class types are lowercase by requirement, **not just by convention — Azure rejects anything else.

- **Tags may stay readable. **Tag values are display metadata and may use uppercase (e.g. PROD); only resource names are normalized. The environment letter in a name is therefore lowercase (p) even though its tag value is PROD.

- **Entra ID objects are out of scope for this rule. **Directory object display names (Section 7) follow their own conventions and may use mixed case; the lowercase rule applies to Azure resource names.

**Readability note: **lowercasing removes the visual word boundaries that mixed case would provide (myappsecondt vs MyappSecondT). This is an accepted trade-off — separators carry readability where they are present, and the canonical lowercase form is what guarantees stable, collision-free behavior across Azure and tooling.

# 9. Practical Deployment Examples

## 9.1 Hub-Spoke Architecture

In a hub-spoke model the hub hosts shared connectivity services and each spoke hosts a workload. The hub is treated as its own application.

### Hub

| **Scope / resource** | **Example name** |
| --- | --- |
| Subscription | hub-sub |
| Resource group (network) | hubp-net-rg |
| Virtual network | hubp-net-net01 |
| Gateway subnet (reserved) | GatewaySubnet |
| VPN gateway | hubp-net-vgw01 |
| Peering | hubp-net-peer-myappsecondt |

### Spoke (Myapp / Second / Test)

| **Scope / resource** | **Example name** |
| --- | --- |
| Subscription | myappsecondt-sub |
| Resource group (SQL) | myappsecondt-sql-rg |
| Virtual network | myappsecondt-net-net01 |
| SQL node VM | myappsecondt-sql-node01 |
| NIC of that node | myappsecondt-sql-node01-n01 |
| Storage (Terraform state) | myappsecondtxdtxtf |
| Deployment identity | myappsecondt-sec-deploy |

## 9.2 New Application Landing Zone

For a newly onboarded application called Yourapp (first instance, production), applying Variant B consistently:

| **Scope / resource** | **Example name** |
| --- | --- |
| Subscription | yourappfirstp-sub |
| Resource group (web) | yourappfirstp-web-rg |
| Virtual network | yourappfirstp-net-net01 |
| App service | yourappfirstp-web-portal |
| Key Vault | yourappfirstp-sec-kv01 |
| SQL server | yourappfirstp-db-sql01 |

# 10. Naming & Tagging Alignment

Naming and tagging are complementary. Naming encodes the minimal critical identity directly in the resource name; tagging carries everything that deliberately does not belong in the name — including the region, the resource type metadata, ownership, and billing. This is the same “only what is needed” principle applied across both mechanisms.

Because this convention deliberately omits the region and the resource type from the name, those facts are recovered from the resource's own properties and from tags. The mandatory tag set below therefore carries information that other conventions would have duplicated into the name.

| **Tag** | **Required** | **Description** | **Example** |
| --- | --- | --- | --- |
| Environment | Yes | Lifecycle stage; matches the environment component | PROD, TEST, DEV |
| Application | Yes | Owning application / workload | Myapp |
| Owner | Yes | Responsible team or person | Platform Team |
| CostCenter | Yes | Cost center for chargeback | CC-12345 |
| Region | Recommended | Deployment region (omitted from name) | westeurope |
| ManagedBy | Recommended | Tool managing the lifecycle | Terraform |

**Convention: **tag values use uppercase for human readability (e.g. PROD), while the environment component in the name uses a single lowercase letter (e.g. p). Both refer to the same environment. The contrast is intentional: tags are free-form display metadata, whereas the name is normalized to lowercase (see Section 8.7).

# 11. Governance, Validation & Compliance

## 11.1 Governance Principles

The naming convention is a governance standard. It applies to all deployments, whether performed through the portal, through infrastructure-as-code (Terraform, Bicep, ARM), or through CI/CD pipelines. Every team deploying resources is responsible for compliance.

Naming supports the wider governance model: it makes resource group boundaries predictable (helping RBAC), it enables pattern-based Azure Policy rules, and it provides meaningful names in alerts, dashboards, and cost reports. Enforcement combines preventive controls (naming logic in Terraform modules, CI/CD validation against regex patterns) with detective controls (Azure Policy audits, periodic Resource Graph reviews).

## 11.2 Abbreviation Registry

**Abbreviations must not be invented ad-hoc. **When a component must be shortened to fit a length limit — most often a layer or resource_name for a storage account, Key Vault, or Windows hostname — the short form must come from a central abbreviation registry maintained alongside this convention. This guarantees the same word is always abbreviated the same way across all teams and environments.

**Layer abbreviations are the most important to standardize. **The layer appears in almost every name and drives how resources group together, so an inconsistent layer set (one team writing db, another data, another sql) fragments the whole estate. Agree one short set up front and register it. A typical starting set:

| **Layer** | **Meaning** | **Typical contents (various resource types)** |
| --- | --- | --- |
| fe | Frontend / presentation | App Service, static web app, front-end VMSS, CDN |
| be | Backend / compute | VMs, disks, NICs, function apps, container apps |
| db | Database | SQL server + DBs, Cosmos, Redis, PostgreSQL |
| dt | Data | Storage accounts, data lakes, Data Factory sinks |
| net | Network | VNet, subnets, NSG, route tables, gateway, bastion |
| sec | Security / identity | Key Vault, managed identities, security tooling |
| mon | Monitoring | Log Analytics, App Insights (where local) |
| int | Integration | Service Bus, Event Hub, API Management, ACR |

**A layer is not a resource type. **This is a common confusion: the layer groups resources by their role in the application, and a single layer normally contains *several different** Azure **types* (the be layer holds VMs, their disks, and their NICs; the net layer holds the VNet, its subnets, the NSG and the gateway). The layer therefore tells you *which** part **of the application* a resource belongs to, not *what kind of** Azure **resource* it is — the type is read from Azure itself (Section 2). The set above is an example, not a fixed list; standardize the layers that fit your estate, but standardize them once.

If a word has no established abbreviation yet, the team proposes one and gets it approved before deployment, then it is added to the registry. The registry is especially important for the application_layer and resource_name components, since these are the ones most often shortened. A consistent registry is also what makes the optional naming policy (Section 11.4) able to validate the layer against a known value set.

**The registry also covers two cases where this convention reuses standardized codes: **(1) the resource type abbreviation used as a resource_name fallback (Section 3.1) — prefer the official CAF abbreviation, and only register a custom one where CAF has none; and (2) the region code used in the rare region-in-name exception (Section 8.5). In both cases, official codes are used where they exist, and any custom code is recorded in the registry so it is spelled consistently everywhere. Official sources: learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations and learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules.

## 11.3 Exception Management

Some situations make strict compliance impossible — auto-generated names, third-party tool limitations, legacy migrations. These follow a formal exception process: documented justification, review, and recording in an exception register. Auto-generated names (Section 8.6) are a standing accepted exception governed through tagging.

## 11.4 Naming Enforcement: Generate vs Verify

**Enforcement has two distinct jobs that must not be confused: ***generating* the correct name when a resource is created, and *verifying* that whatever was created complies. These belong to two different tools, and the verify side has a hard practical limit worth stating up front.

- **Generation belongs to the IaC layer **(Terraform / Bicep). The name is composed deterministically from the components, applying the inheritance logic from Section 3. This is where “the right name appears automatically” actually happens.

- **Verification belongs to Azure Policy **(audit). Policy does not build names — it only reports whether what was created starts with the correct inherited prefix.

### Generate names in IaC, as shared code

**If you deploy with infrastructure-as-code, naming should not be retyped per resource or per module. **It belongs in one place — a shared naming function / library that every module calls — so the convention is applied identically everywhere and a change to the rules propagates from a single source. Human-typed names drift; code-generated names do not. The library takes the components and the chosen scope mapping and returns the name; callers never assemble names by hand.

### IaC naming module (pseudocode)

A minimal shape of that shared function: it receives the components and the chosen scope mapping, assembles subscription, resource group, and resource names so each inherits its parent's prefix, and normalizes the whole name to lowercase as the final step. Inputs may be written readably (mixed case); the output is always lowercase (see Section 8.7).

```
# inputs (may be written readably; case is normalized below)
application   = "Myapp"
instance      = "Second"
environment   = "T"          # d | t | p
layer         = "Sql"
resource_name = "Node"
resource_seq  = "01"

# scope mapping (how many components the sub/RG absorb)
sub_prefix = "${application}${instance}${environment}"   # Variant B
rg_name    = "${sub_prefix}-${layer}-rg"
res_name   = "${sub_prefix}-${layer}-${resource_name}${resource_seq}"

# storage-class types: x separator, no hyphens
is_storage = contains(["storage","acr"], type_class)
raw = is_storage
    ? replace("${sub_prefix}x${layer}x${purpose}${seq}", "-", "")
    : res_name

# CASE NORMALIZATION — always lowercase, for every type
# (Azure is case-insensitive and may echo names back in any case;
#  a single canonical form avoids Terraform/state drift)
name = lower(raw)

# length guard: shorten via the abbreviation registry, never the prefix
if length(name) > max_len[resource_type] {
    name = lower(shorten_via_registry(name))   # see Section 11.2
}
```

### Verification policy: audit only

**Because of the exceptions, this policy is realistically an audit tool, not a gate. **Azure constantly creates resources you do not name — NICs, OS disks, AKS node resource groups, child resources, deployment objects — and new resource types appear over time. A deny policy broad enough to cover the resources you *do* control will inevitably also catch some you do *not*, and block a legitimate deployment. Run the policy in audit so it reports non-compliance without ever blocking; treat its output as a report to act on, not a barrier.

The design goal is the same as before — **act where you have naming freedom, stay silent where you do not** — achieved through what the policy matches rather than through the effect:

- **Scope (subscription + resource group). **Stable shape, fully under your control; the most valuable thing to keep clean since everything inherits from it.

- **A narrow allowlist of backbone resource types. **Only the types whose names you genuinely own and whose shape is predictable.

- **Silent on everything else. **Auto-generated names, child resources, and any unlisted or new type are simply not matched — no noise.

**Why allowlist and not blocklist: **“where I have no freedom” is an open, growing set — auto-generated names depend on who created the resource (not its type), child types are many, and new Azure types appear constantly. A blocklist would default unknown types to *flagged*, so each new service raises noise until someone adds an exception. An allowlist defaults to *silent*: its only risk is a missed check (cheap — fix later). For naming governance, allowlist is the safer default. If you ever promote any rule to deny, limit it strictly to the scope rule (sub/RG) and only after the audit has been quiet on legitimate deployments for a while.

Rule 1 — scope audit (subscription + resource group):

```json
{
  "mode": "All",
  "policyRule": {
    "if": { "anyOf": [
      { "allOf": [
        { "field": "type", "equals":
            "Microsoft.Resources/subscriptions" },
        { "field": "name", "notMatch": "*-sub" }
      ] },
      { "allOf": [
        { "field": "type", "equals":
            "Microsoft.Resources/subscriptions/resourceGroups" },
        { "field": "name", "notLike": "*-rg" },
        { "field": "name", "notMatch":
            "[concat(substring(field('subscriptionDisplayName'), 0,
              length(field('subscriptionDisplayName')) - 4), '*-rg')]" }
      ] }
    ] },
    "then": { "effect": "audit" }
  },
  "displayName": "Naming — scope (sub/RG) [audit]"
}
```

Rule 2 — backbone resource audit (allowlist + RG-prefix inheritance):

```json
{
  "mode": "All",
  "policyRule": {
    "if": { "allOf": [
      { "field": "type", "in": [
          "Microsoft.Compute/virtualMachines",
          "Microsoft.Compute/disks",
          "Microsoft.Network/virtualNetworks",
          "Microsoft.Network/networkSecurityGroups",
          "Microsoft.Network/routeTables",
          "Microsoft.Network/publicIPAddresses",
          "Microsoft.KeyVault/vaults",
          "Microsoft.Storage/storageAccounts",
          "Microsoft.Sql/servers",
          "Microsoft.ManagedIdentity/userAssignedIdentities" ] },
      { "field": "name", "notMatch":
          "[concat(field('resourceGroupName'), '*')]" }
    ] },
    "then": { "effect": "audit" }
  },
  "displayName": "Naming — backbone resources [audit]"
}
```

**Notes to tune: **the Rule 2 prefix check assumes the resource inherits the full RG name (the hyphenless storage form will not match a hyphenated RG name — give storage its own rule, or exclude it from this audit and check it separately). The allowlist is deliberately short; extend it only with types whose names you fully control. Both rules run in audit; review the compliance results periodically and feed corrections back into the IaC naming library rather than promoting the policy to deny.

**Apply: **Azure portal → Policy → Definitions → + Policy definition → paste each JSON → assign to the desired scope (ideally a management group), both in audit.
