---
sidebar_position: 1
---

# Introduction

**Metadata Management Module** (MMM) is a platform for building Ataccama ONE Gen2.

Ataccama owns quite a lot of products (Data catalog, Data quality management, Reference data management, Master data management, and more). Some of them run on different platforms. The ultimate long-term goal is to integrate all Ataccama products to Ataccama ONE Gen2.

<aside>
💡 What the heck is meta-meta-data? Ataccama products collects and manages meta-data about client's data (e.g. how many rows are there in a particular DB table). The shape of this meta-data is configurable. And that configuration is called 🥁 ... meta-meta-data! So it's metadata describing metadata that Ataccama holds about client's data.

</aside>

# MMM

MMM is a highly configurable web application extendible by plugins. It is meta-meta-data (MMD) driven. MMD serves as a domain model and the whole application (database model, GraphQL schema, FE user interface, ...) is generated from it.

BE (Java) and FE (Typescript) communicates using GraphQL. Both BE and FE are consists of MMM Core and MMM Plugins.

MMM Core is basically a framework providing a generic functionality and generic UI on FE. Plugins extending this functionality are picked up and ran by MMM Core.

MMM also uses other modules within the Ataccama infrastructure. Either to extend the MMM features (e.g. AI module) or to integrate other Ataccama products (e.g. RDM module).

## MMM FE

The Core of MMM FE generates all the create/edit/detail/listing screens based on the MMD provided by the BE. And more! It creates a navigation, handles routing and permissions, and other...

FE plugins provides a way to override/enhance the generic functionality that's provided by Core. Plugins can target concrete entities/screens that are essential for the ONE Gen2 (like a database connection or a glossary term) which are expected to be present. Or can target entities based of a particular shape or behaviour (trait).

[Getting started](https://www.notion.so/Getting-started-0e12ed1703674ceab86893bd238ccba6) — let's do this 😎
