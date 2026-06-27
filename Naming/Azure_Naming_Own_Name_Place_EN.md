# Azure Naming: Own Name Place

**Logical Name in a Tag and Technical ID for Azure Resources**

This document is a companion to `Azure_Naming_Principles_EN.md` and `Azure_Naming_Convention.docx`. It works out the "Own Name Place" approach introduced there: a **logical name stored in a tag** plus a **short technical ID** used as the Azure resource name.

---

## 1. Purpose

`Azure_Naming_Principles_EN.md` established that, from the resource-group level downward, the Azure resource "name" is structurally part of the Resource ID, and therefore cannot reliably act as a human-readable name. It identified two ways forward and deferred their detailed design:

- a separated ID and a hierarchical name stored in a tag ("Own Name Place");
- hierarchical naming directly in the resource name ("Name-Based Application Model").

This document works out the **first option in full**: both halves of it — the logical name in the tag, and the technical ID used as the Azure resource name — with equal weight, since a usable model needs both, not just the ID.

The guiding principle from the principles document carries over unchanged:

> **A name is a carrier of meaning. An ID is a carrier of identity.**
> The Azure resource name plays the role of the ID. The `Name` tag plays the role of the name.

---

## 2. Design Principle

| | Technical ID (Azure resource name) | Logical name (`Name` tag) |
|---|---|---|
| Primary purpose | unambiguous identification | readability, orientation, communication |
| Audience | systems, automation | people |
| Stability | fixed, never changes | can evolve if the workload structure changes |
| Content | resource type + sequence + org discriminator only | full workload hierarchy |
| Constrained by | Azure per-type length/charset limits | tag value length only (256 chars) |

The two halves are deliberately decoupled: the ID does not need to be readable, and the name does not need to be short. Each is optimized for what it actually has to do, which is the core argument made in the principles document against the relaxed-CAF approach (Approach 1), where a single name tries — and fails — to serve both purposes at once.

---

## 3. Logical Name (stored in the `Name` tag)

### 3.1 Why a Tag, Not the Resource Name

The principles document's workload model describes a system as a hierarchy: **Application → Application instance → Layer → Component → Component part**. A name that reflects this hierarchy needs:

- a consistent separator between every part;
- no shortening forced by platform character limits (storage accounts, Key Vault, Windows hostnames, …);
- the ability to change without breaking automation that depends on resource identity.

Azure resource names cannot guarantee any of this once they become part of the Resource ID. The tag can, because the tag is metadata, not identity. Storing the logical name in a tag is therefore not a workaround — it is the direct consequence of accepting that, in Azure, the resource name is the ID.

### 3.2 Tag Key

Recommended tag key:

```text
Name
```

Acceptable alternatives if an organization already has an established tagging standard (must be standardized — do not mix multiple keys for the same purpose):

```text
LogicalName
DisplayName
WorkloadName
```

### 3.3 Structure

The logical name reuses the workload model from the principles document directly, mapped one-to-one onto name components:

| Workload model (principles doc) | Name component | Example |
|---|---|---|
| Application | `application` | `myapp` |
| Application instance | `instance` | `second` |
| — | `environment` | `test` |
| Layer | `layer` | `sql` |
| Component | `component` | `node` |
| Component instance | `number` | `01` |
| Component part (optional) | `part` | `nic` |
| Component part instance (optional) | `number` | `01` |

Standard form:

```text
<application>-<instance>-<environment>-<layer>-<component>-<number>
```

With an optional dependent component part:

```text
<application>-<instance>-<environment>-<layer>-<component>-<number>-<part>-<number>
```

Examples:

```text
myapp-second-test-sql-node-01
myapp-second-test-sql-node-01-nic-01
myapp-second-test-sql-node-01-disk-01
myapp-second-test-net-vnet-01
myapp-second-test-sec-keyvault-01
```

This follows the hierarchical-structure rule from the principles document directly: **more general information on the left, more specific information on the right**, so that related objects sort and group together naturally.

### 3.4 Separator

Because the value lives in a tag and never has to satisfy a resource-type-specific charset, the convention can — and should — use a **single consistent separator (`-`) between every component**, with no exceptions and no fallback rules. This is the opposite of the resource-name-based convention, where the separator sometimes has to be dropped because a resource type forbids hyphens.

### 3.5 Minimizing Redundancy — and the Resource Type Token

Per the principles document's rule that a name should not repeat information available elsewhere in context:

- Do not add `Region` into the logical name — it belongs in the `Region` tag.
- Do include the components that are *not* derivable from anywhere else: application, instance/environment, layer, component, and (where relevant) component part — exactly the workload hierarchy, nothing more.

The Azure resource type, however, is worth revisiting: by itself it is usually recoverable from the resource's `type` property and from the technical-ID prefix, so adding it would normally be redundant. But the *component* slot does not always describe a resource type — it often describes a **functional role** (`node`, `portal`, `gateway`), and a functional role is genuinely not derivable from anywhere else. That is the case where adding the resource-type token earns its place.

Extended structure, with the resource-type token inserted directly before the component:

```text
<application>-<instance>-<environment>-<layer>-<resource_type>-<component>-<number>[-<part>-<number>]
```

Use the same abbreviation list as the technical ID (`vm`, `kv`, `st`, `sql`, …) — no separate list of tokens for tags.

**This is really one rule applied twice, not two separate rules:** a slot is included only if it adds information not available elsewhere.

- If the **component's functional role and the resource type are different things** (`node` vs. `vm`), both slots earn their place: `…-sql-vm-node-01`.
- If the **component name would just repeat the resource type** (`keyvault`, `vnet`, `storage`), keep only one of them — there is no point writing the same word twice: `…-sec-keyvault-01`, not `…-sec-kv-keyvault-01`.
- If the **resource type and index are already sufficient on their own** — i.e. there is no separate functional role worth naming — drop the component slot entirely and use the resource-type token with its index directly. The result is the same name, just without a redundant extra word:

```text
myapp-second-test-net-vnet-01           # resource type doubles as the component name — fine as is
myapp-second-test-sec-kv-01             # resource type + index is enough; no separate component needed
myapp-second-test-sql-vm-node-01        # functional role ("node") adds real information → keep both
```

In short: name every slot that adds information, and only that.

### 3.6 Readability Check

Per the principles document's trade-off ("more structure → worse readability, less structure → less information"), the logical name should stay readable at a glance without needing a lookup table. As a practical test: if a teammate unfamiliar with the resource can read the tag value and correctly state *application*, *environment*, and *role* without opening documentation, the structure is working.

### 3.7 Required and Recommended Tags

| Tag | Required | Purpose | Example |
|---|---|---|---|
| `Name` | Yes | Full logical name (hierarchical, human-readable) | `myapp-second-test-sql-node-01` |
| `Application` | Yes | Owning application/workload | `Myapp` |
| `ApplicationInstance` | Recommended | Instance, deployment, tenant, or variant | `Second` |
| `Environment` | Yes | Lifecycle stage | `TEST` |
| `Layer` | Recommended | Logical application/infrastructure layer | `sql` |
| `Owner` | Yes | Responsible team or person | `Platform Team` |
| `CostCenter` | Yes | Cost allocation | `CC-12345` |
| `Region` | Recommended | Deployment region | `westeurope` |
| `ManagedBy` | Recommended | Lifecycle tool | `Terraform` |
| `TechnicalId` | Optional | Explicit copy of the Azure resource name, for cross-checking in reports | `vm0001234acm` |

Storing both the single aggregated `Name` tag **and** the structured component tags (`Application`, `Environment`, `Layer`, …) gives the best of both worlds: `Name` is what a person reads, the structured tags are what a query filters or groups by.

Microsoft tag rules that constrain this design: tags are plain-text key-value metadata, must not contain sensitive values, and tag values are commonly limited to 256 characters — comfortably enough for the proposed structure under normal circumstances.

### 3.8 Worked Examples

| Azure object | Technical ID | `Name` tag |
|---|---|---|
| Resource group | `rg0000456acm` | `myapp-second-test-sql` |
| Virtual machine | `vm0001234acm` | `myapp-second-test-sql-node-01` |
| NIC | `nic0009871acm` | `myapp-second-test-sql-node-01-nic-01` |
| Disk | `disk0003344acm` | `myapp-second-test-sql-node-01-disk-01` |
| Storage account | `st0009876acm` | `myapp-second-test-data-lake-01` |
| Key Vault | `kv0000045acm` | `myapp-second-test-sec-keyvault-01` |

The Azure resource name tells the operator *which exact resource it is*. The `Name` tag tells them *what it represents* — the same division of labor the principles document draws between name and ID, just made concrete.

---

## 4. Technical ID (Azure resource name)

### 4.1 Core Principle

The Azure resource name SHALL:

- contain **no separators** — only lowercase letters and digits;
- carry **no business meaning** beyond resource type;
- be stable and deterministic;
- work **without a central ID-issuing registry**.

### 4.2 Structure

```text
<resource_type><sequence><org_identifier>
```

**Resource type** — letters only, lowercase. Length is not fixed; use the official Microsoft CAF abbreviation wherever one exists rather than inventing a custom token. Custom tokens must go through the same abbreviation registry used by the naming convention, not be invented ad hoc.

**Sequence** — numeric only, unique per organization **and** per resource type (each resource type keeps its own counter). Default assumption: 7 digits (`0000001`–`9999999`); adjust up if an organization expects more than ~10 million resources of one type, or down if it will stay small.

**Organization identifier** — max 5 characters, alphanumeric, must start with a letter. A pure uniqueness discriminator — not the company name — kept constant across the organization. On collision with another organization's identifier, resolve by using more of the available characters, never by giving it semantic meaning.

No central registry is required: the resource-type abbreviation list is a static reference, the sequence is a local counter per org + resource type, and the org identifier is a fixed constant chosen once.

### 4.3 Length Validation

| Resource | Limit | Note |
|---|---|---|
| Windows VM hostname | **15 characters** | Most restrictive practical limit |
| Storage account | 24 characters, alphanumeric only | Globally scoped, no hyphen |
| Key Vault | 3–24 characters, globally scoped | DNS-based name |
| Most other resource types | 60+ characters | Not a practical constraint |

Worst case at full component length (`4 + 7 + 5 = 16`) overflows the 15-character VM hostname target, but in reality VM is ok - it is 2+7+5 


### 4.4 Mandatory Rules

- Only lowercase letters and digits, no separators.
- Target maximum length: **15 characters**, to cover the Windows VM hostname worst case,  **24 characters** for some other restricted resources
- `resource_type` follows the CAF abbreviation,
- `sequence` unique per org + resource type, 7 digits by default.
- `org_identifier` ≤5 chars, alphanumeric, starts with a letter, constant org-wide, never given semantic meaning.

---

## 5. Working With the Model

### 5.1 Human Operations

Operators read the `Name` tag to understand a resource's purpose; the Azure resource name only confirms exactly which resource it is.

### 5.2 Automation

```text
Automation identifies resources by technical ID.
Humans understand resources by logical name.
```

- Terraform/IaC state keys on the technical ID.
- Azure Policy validates presence and format of required tags.
- Resource Graph queries expose both technical ID and logical name side by side.
- Cost and inventory reports group by the structured tags (`Application`, `Environment`, `Owner`, `CostCenter`).

### 5.3 Example Resource Graph Query

```kusto
Resources
| project
    id,
    type,
    technicalName = name,
    logicalName = tostring(tags['Name']),
    application = tostring(tags['Application']),
    environment = tostring(tags['Environment']),
    owner = tostring(tags['Owner']),
    costCenter = tostring(tags['CostCenter']),
    location
| order by application asc, environment asc, logicalName asc
```

---

## 6. Governance and Validation

Generation (by IaC, not by hand) should: receive the resource type → obtain the next sequence number for that org + type → assemble the technical ID → validate length/charset → write the logical name into the `Name` tag → apply all mandatory tags.

| Area | Validation |
|---|---|
| Technical ID | Format, resource-type token, sequence digits, allowed characters, ≤15 chars |
| Logical name tag | Structure, required components present, separator consistently used |
| Mandatory tags | Presence and allowed values |
| Global resources | Name availability checked before deployment |
| Abbreviation list | Resource-type tokens are registered, not invented ad hoc |

---

## 7. Open Decisions

| Decision | Recommended answer |
|---|---|
| One `Name` tag or several structured tags? | Both — aggregated `Name` tag plus structured tags for reporting. |
| Should the technical ID carry business meaning? | No — only resource type, sequence, and org discriminator. |
| Sequence global or per resource type? | Per resource type, per organization. |
| Default sequence length? | 7 digits; adjust per organization's projected scale. |
| Is a central allocator required? | No — local counters per org + resource type are sufficient. |
| Org identifier length and meaning? | ≤5 chars, alphanumeric, starts with a letter; pure discriminator, not the company name. |
| Target maximum ID length? | 15 characters, to safely cover the Windows VM hostname constraint. |
| Should tags contain sensitive data? | No — tags are plain-text metadata. |

---

## 8. Final Recommendation

```text
Technical ID (Azure resource name):
<resource_type><sequence><org_identifier>

Logical name (Name tag):
<application>-<instance>-<environment>-<layer>-<component>-<number>[-<part>-<number>]
```

| Azure object | Technical ID | `Name` tag |
|---|---|---|
| Resource group | `rg0000456acm` | `myapp-second-test-sql` |
| Virtual machine | `vm0001234acm` | `myapp-second-test-sql-node-01` |
| NIC | `nic0009871acm` | `myapp-second-test-sql-node-01-nic-01` |
| Disk | `disk0003344acm` | `myapp-second-test-sql-node-01-disk-01` |
| Storage account | `st0009876acm` | `myapp-second-test-data-lake-01` |
| Key Vault | `kv0000045acm` | `myapp-second-test-sec-keyvault-01` |

This is the full implementation of the "Own Name Place" approach from `Azure_Naming_Principles_EN.md`: the Azure resource name stays short, separator-free, and automation-friendly; the `Name` tag stays readable, hierarchical, and unconstrained by platform-specific naming rules — and neither half is forced to compromise to serve the other's purpose.
