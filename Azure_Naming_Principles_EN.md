# Azure Naming Principles

**Name, Identifier and Naming Strategy in Azure**

*Working document*

## Table of Contents

- [Purpose of This Document](#purpose-of-this-document)
- [Name vs. Identifier (ID)](#name-vs-identifier-id)
  - [Definition](#definition)
  - [Basic Principle](#basic-principle)
  - [Properties](#properties)
  - [Carrying Information](#carrying-information)
  - [Usage](#usage)
  - [Key Rules](#key-rules)
  - [Summary](#summary)
- [Naming and Identifiers in Azure](#naming-and-identifiers-in-azure)
  - [Basic Principle](#basic-principle-1)
  - [Resource ID Structure](#resource-id-structure)
  - [Important Fact](#important-fact)
  - [Overview of the Role of Name vs ID](#overview-of-the-role-of-name-vs-id)
  - [Interpretation](#interpretation)
- [What CAF Says](#what-caf-says)
  - [Interpretation of the CAF Approach](#interpretation-of-the-caf-approach)
  - [An Important Platform Aspect](#an-important-platform-aspect)
  - [Observations from Practice](#observations-from-practice)
  - [Impact on Convention Design](#impact-on-convention-design)
  - [Summary](#summary-1)
- [Freedom of Naming and What It Should Contain](#freedom-of-naming-and-what-it-should-contain)
  - [Basic Principle](#basic-principle-2)
  - [Hierarchical Structure](#hierarchical-structure)
  - [Minimizing Redundancy](#minimizing-redundancy)
  - [Readability as a Key Requirement](#readability-as-a-key-requirement)
  - [A View of IT Systems (Workload Model)](#a-view-of-it-systems-workload-model)
  - [Impact on Naming](#impact-on-naming)
  - [Summary](#summary-2)
- [Options in Azure](#options-in-azure)
  - [Starting Situation](#starting-situation)
  - [Approach 1: Relaxed CAF Interpretation](#approach-1-relaxed-caf-interpretation)
  - [Approach 2: Own Name Place (Name in Tag)](#approach-2-own-name-place-name-in-tag)
  - [Approach 3: Name-Based Application Model](#approach-3-name-based-application-model)
  - [Summary](#summary-3)
  - [Note](#note-1)
- [How to Continue](#how-to-continue)
  - [Separating the ID and the Name (Own Name Place)](#separating-the-id-and-the-name-own-name-place)
  - [Name-Based Application Model](#name-based-application-model)
  - [Next Steps](#next-steps)
- [References](#references)
  - [Étienne Brouillard – “AWS Resources Shouldn’t Be Named”](#tienne-brouillard-aws-resources-shouldnt-be-named)
  - [Microsoft Cloud Adoption Framework (CAF)](#microsoft-cloud-adoption-framework-caf)
  - [Hybrid Naming & Tagging Strategy (DataChefHQ)](#hybrid-naming-tagging-strategy-datachefhq)
  - [Application Naming & UID Approach (APM / Enterprise IT)](#application-naming-uid-approach-apm-enterprise-it)
  - [Azure Resource Naming Convention: Strategies and Examples (Paladin Cloud)](#azure-resource-naming-convention-strategies-and-examples-paladin-cloud)

## Purpose of This Document

The purpose of this document is to explain the relationship between the concept of a name, an identifier, and a practical naming strategy in the Microsoft Azure environment. The document describes why a resource name in Azure cannot always be understood purely as a human-readable label, and why in many cases the name effectively becomes part of the technical identifier.

The document serves as a foundation for designing a consistent naming convention that takes into account both the recommendations of the Microsoft Cloud Adoption Framework and the practical constraints of the Azure platform. It focuses in particular on the distinction between readability, unambiguous identification, name stability, and the ability to carry meaningful information in the name or in metadata.

The result of this document is not a single universal naming convention, but rather a framework for deciding which approach to adopt in a given environment. The document compares several possible models and describes their impact on management, automation, operational readability, and the long-term sustainability of the Azure environment.

## Name vs. Identifier (ID)

### Definition

A name is a human-readable label used for communication and orientation. An identifier (ID) is a unique identifier used to identify an object within a system.

### Basic Principle

A name says “what it is called”; an ID says “what it actually is”.

### Properties

| Area | Name | Identifier (ID) |
| --- | --- | --- |
| Primary purpose | readability and communication | unambiguous identification |
| Target audience | human | system |
| Uniqueness | not required | mandatory |
| Stability | can change | does not change |
| Readability | high | often low |

### Carrying Information

| Area | Name | Identifier (ID) |
| --- | --- | --- |
| Content of meaning | common | limited or none |
| Type of information | flexible | fixed |
| Change of meaning | possible | requires a new ID |
| Interpretation | context-dependent | unambiguous |

### Usage

| Scenario | Name | Identifier (ID) |
| --- | --- | --- |
| Communication between people | yes | no |
| Operations / troubleshooting | yes | to a limited extent |
| Automation / integration | no | yes |
| Unambiguous reference | no | yes |

### Key Rules

- A name can carry meaning, because it can change.
- An ID must remain stable, and therefore cannot contain meaning that changes.
- An ID is the source of truth for identity.
- A name is optimized for readability and orientation.

### Summary

A name is a carrier of meaning. An ID is a carrier of identity.

## Naming and Identifiers in Azure

### Basic Principle

In Azure, every resource is identified using a Resource ID with a fixed structure.

### Resource ID Structure

```
/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/<provider>/<type>/<resourceName>
```

### Important Fact

A Resource ID is composed of parts that play different roles. Some parts are IDs, others are referred to as a name, but in practice function as an identifier.

### Overview of the Role of Name vs ID

| Level | Name and ID separated | ID type | Role of the name | Impact on naming | Assessment |
| --- | --- | --- | --- | --- | --- |
| Management Group | Yes | Standalone ID | Readable label only | Full design freedom | No issues |
| Subscription | Yes | Standalone (GUID) | Readable label only | Full design freedom | No issues |
| Resource Group | No | Composite (part of Resource ID) | Part of identification | Limited freedom (uniqueness, length, changes) | Constrained |
| Resource | No | Composite (part of Resource ID) | Part of identification | Limited freedom (syntax, length, change) | Constrained |
| Child Resources | No | Composite | Often generated | Minimal control over naming | Significantly constrained |

### Interpretation

Management Group and Subscription preserve the separation of name and ID. From Resource Group downward, the name becomes part of the ID and constrains the design.

**Conclusion: From the Resource Group level downward, the name cannot be separated from the ID.**

## What CAF Says

The Cloud Adoption Framework (CAF) recommends establishing a consistent naming convention for Azure resources.

According to the official documentation:

- a naming convention should provide “standardized formats” for naming resources
- each organization should define its own convention that fits its needs
- CAF lists recommended components, such as resource type, workload / application, environment, region, and instance

A typical example used in the CAF documentation has the form:

```
<resource-type>-<workload>-<environment>-<region>-<instance>
```

This example illustrates one possible interpretation, not a binding rule.

### Interpretation of the CAF Approach

From this example it follows that:

- the name is designed as a combination of identifying information (instance, uniqueness) and meaningful information (type, environment, region)

The CAF sample model treats the name as a combination of both name and ID components.

At the same time, CAF explicitly states that names must meet requirements for uniqueness, length, and allowed characters, and that most names are immutable after creation.

### An Important Platform Aspect

In Azure there is a group of resources whose names are either generated by the platform or are heavily constrained by the rules of the specific resource type.

CAF also states that not all resources can use the same naming pattern.

### Observations from Practice

As a result of these constraints, there are names that cannot be defined according to the chosen convention or that are generated automatically.

These names are not always consistent across resource types and are not harmonized with one another.

### Impact on Convention Design

It cannot be relied upon that all names in Azure will fulfil the role of a name in the sense of a readable label.

In these cases it is safer to proceed from the principle that a name is primarily an identifier (part of the ID), and any meaning it contains is merely supplementary.

### Summary

CAF provides principles, recommended components, and illustrative examples – not a strict standard.

Given the characteristics of the Azure platform, it is necessary to treat names primarily as identifiers and to regard any meaning they contain as secondary.

## Freedom of Naming and What It Should Contain

### Basic Principle

When designing names in any environment, the goal is to create a label that:

- allows a person to orient quickly
- preserves the logical structure of the system
- does not carry unnecessary or redundant information

### Hierarchical Structure

Names should be hierarchical.

This means that:

- parts of the name reflect the structure of the system
- more general information is placed on the left
- more specific information is placed on the right

for example:

```
<application><instance><environment>-<layer>-<component>
```

#### Reason

Related objects should be close to one another not only in structure but also in name.

Consequences:

- easier orientation in a list of resources
- natural grouping when sorting
- quick identification of relatedness

### Minimizing Redundancy

A name should not contain information that is:

- available elsewhere in context
- unambiguously derivable

for example:

- there is no need to repeat the region if it is stated alongside
- there is no need to repeat the type if it is obvious from the context

#### Conversely

A name should contain:

- information that is not available elsewhere
- information that is needed for orientation

A name supplements the context; it does not duplicate it.

### Readability as a Key Requirement

Regardless of structure, the following holds:

- a name must remain readably recognizable by a human
- a name must be interpretable without complex logic

Trade-off:

- more structure → worse readability
- less structure → less information

The goal is not to maximize information, but to preserve readability.

### A View of IT Systems (Workload Model)

For the purposes of a naming convention, it is useful to view IT systems as workloads.

#### Structure of a Workload

A workload can be divided into:

1. **Application**
   - represents a logical unit (a service, a system)
2. **Application instance**
   - multiple variants of one application, for example different consumers or separate configurations
   - or environments (dev, test, prod)
3. **Layers**
   - frontend
   - backend
   - data
4. **Components**
   - individual parts of a layer
   - for example services, databases, or integrations
5. **Component parts**
   - detailed elements
   - instances
   - dependencies
   - supporting objects

### Impact on Naming

From this model it follows that:

- the name should reflect this structure
- the name should preserve its order

*Naming is not an arbitrary label. It is a projection of the workload's structure into the name.*

### Summary

- names should be hierarchical
- they should group related objects
- they should not duplicate available information
- they should preserve readability
- they should reflect the structure of the workload

*A good name does not describe everything. It describes what would otherwise not be visible.*

## Options in Azure

### Starting Situation

As shown in the previous chapters:

- in Azure:
  - names are not separated from identifiers
  - the resource “name” is part of the Resource ID

In Azure, therefore, we do not actually work with pure names, but with identifiers.

This fact limits the freedom to design a naming convention and makes it necessary to choose a specific approach.

### Approach 1: Relaxed CAF Interpretation

#### Characteristics

- based on the CAF example
- tries to embed in the name:
  - type
  - application
  - environment
  - region
  - instance
- if the rules do not allow the name to be created:
  - the information is shortened
  - or it is omitted

#### Advantages

- simple to understand
- close to Microsoft's recommendation
- relatively readable

#### Disadvantages

- inconsistency (different for each resource)
- collisions with limits (length, charset)
- loss of information when “shortening”
- names are not deterministic

### Approach 2: Own Name Place (Name in Tag)

#### Characteristics

- separation of:
  - name → ID
  - the “Name” tag → the actual name
- minimized ID
- the name contains the workload hierarchy

#### Note

This approach can be implemented either using multiple tags or a single aggregated field (Name).

From a design perspective, this is the same principle – separating identity and meaning.

#### Advantages

- eliminates renaming
- allows full, meaningful names
- clean separation

#### Disadvantages

- dependency on tags
- poorer usability outside Azure
- tooling must use the tag

### Approach 3: Name-Based Application Model

#### Characteristics

- the name contains the workload hierarchy
- uses a prefix and structure

#### Example

```
<application><instance><environment>-<component>
```

#### Advantages

- natural grouping
- good readability
- works outside Azure as well

#### Disadvantages

- Azure limits
- inconsistency for some resources
- risk of renaming

### Summary

There is no universal solution.

The approaches optimize for:

- CAF → simplicity
- Own Name Place → separation of identity
- Name-based → readability

*The choice is a trade-off.*

### Note

In practice, a combination of the name-based approach and tags is used.

## How to Continue

The previous chapters have shown:

- that Azure does not work with full-fledged names
- that the resource “name” is part of the identifier
- that an approach based on the CAF example has its limitations

**For these reasons, this document will not further pursue a naming convention design based on the CAF example or its variants.**

Instead, it focuses on two approaches that better match the characteristics of the platform:

### Separating the ID and the Name (Own Name Place)

An approach based on:

- separating:
  - the technical identifier (ID)
  - the logical name
- using:
  - the ID for identification
  - a tag (e.g. “Name”) to store the actual name

In this model:

- the ID:
  - is stable
  - carries minimal information
  - does not change
- the name:
  - contains the workload hierarchy
  - is optimized for readability

### Name-Based Application Model

An approach based on:

- attempting to project:
  - the workload hierarchy
  - the structure of the system

directly into the resource name.

In this model:

- the name:
  - represents the application, its instance, and structure
  - forms a direct reflection of the architecture

### Next Steps

A detailed design of both approaches:

- a separated ID and a hierarchical name stored in a tag
- hierarchical naming directly in the resource name

**will be further developed in separate documents.**

**The goal is to choose an approach that respects the constraints of Azure while preserving readability and comprehensibility.**

## References

This chapter does not provide a classic, neutral bibliography. Instead, each source is briefly evaluated and explicitly related back to the model proposed in this document – where the two views agree, and where they diverge.

### Étienne Brouillard – “AWS Resources Shouldn’t Be Named”

[View article](https://www.rainmaking.cloud/articles/usetagsinsteadofresourcenames)

**Main idea**

The author questions the value of complex naming conventions and proposes to:

- minimize the meaning carried by the name
- use tags as the primary source of information

The argument rests mainly on the claims that:

- naming is not effective for search
- complex names are difficult to maintain
- metadata (tags) provides a better structure

**Relationship to this proposal**

Agreement:

- meaning does not primarily belong in the name
- a naming convention has limited value
- metadata (tags) is a more suitable carrier of meaning

Difference:

- this approach:
  - effectively gives up on the readability of the name
- the model proposed in this document:
  - preserves readability, but moves it outside the ID

*This article can be understood as an extreme variant of the “Own Name Place” approach.*

---

### Microsoft Cloud Adoption Framework (CAF)

[Define your naming convention](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)

**Main idea**

CAF recommends:

- using structured naming conventions
- combining in the name:
  - type
  - workload
  - environment
  - region
  - instance

**Relationship to this proposal**

Agreement:

- an emphasis on consistency
- an acknowledgment of platform limitations
- a recommendation to use tags for supplementary information

Difference:

- CAF implicitly assumes:
  - that the name is a carrier of meaning
- this proposal explicitly identifies:
  - that in Azure the name is part of the ID
  - and is therefore unsuitable as the primary carrier of meaning

*This proposal addresses a conflict that CAF does not cover: the conflict between the name and the identifier.*

---

### Hybrid Naming & Tagging Strategy (DataChefHQ)

[View repository](https://github.com/DataChefHQ/cloud-naming-strategy/blob/main/README.md)

**Main idea**

Proposes a combined approach:

- a short name containing basic identification
- context extended through tags

**Relationship to this proposal**

Agreement:

- not all information belongs in the name
- tags supplement naming
- the necessity of adapting to cloud provider limits

Difference:

- the hybrid model:
  - still relies on meaning being carried in the name
- this proposal:
  - clearly separates the ID (name) from the meaning (tag / logical name)

*This proposal pushes the hybrid approach further, toward a clean separation of identity and meaning.*

---

### Application Naming & UID Approach (APM / Enterprise IT)

[View article](https://www.sparxsystems.us/application-portfolio-management/application-naming-convention-guide/)

**Main idea**

In enterprise IT, a combination of the following is commonly used:

- a human-readable name
- a unique identifier (UID)

**Relationship to this proposal**

Agreement:

- separation of:
  - the name
  - the ID
- the ID resolves uniqueness
- the name resolves communication

Difference:

- classic enterprise IT:
  - assumes the name is a standalone entity
- Azure:
  - the name = part of the ID

*The proposed approach adapts the classic name + ID model to an environment where this separation does not natively exist.*

---

### Azure Resource Naming Convention: Strategies and Examples (Paladin Cloud)

[View article](https://paladincloud.io/azure-security-best-practices/azure-resource-naming-convention/)

**Main idea**

This practitioner-oriented article takes a deliberately pragmatic, middle-ground position:

- names should carry information that makes a resource easily identifiable at a glance
- tags carry information that adds operational value and supports filtering
- combining naming conventions with tags maximizes the benefit of both, rather than choosing one over the other

**Relationship to this proposal**

Agreement:

- names and tags serve different purposes and should not duplicate the same information
- tags are essential for operational filtering, cost reporting, and incident response

Difference:

- this article:
  - treats the choice between naming and tagging mainly as a tooling and automation question, without questioning whether the platform-level coupling of name and ID is itself the root cause
- this proposal:
  - argues that because the name is structurally part of the Resource ID in Azure, the question is not only “naming vs. tagging” but “identity vs. meaning”

*This source is valuable mainly as evidence that, in practice, most teams converge on a pragmatic combination of naming and tagging – without resolving, or even raising, the underlying identity/meaning conflict that this document addresses explicitly.*

