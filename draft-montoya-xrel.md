---
title: Extended Link Relationships
abbrev: xrel
docname: draft-montoya-xrel-latest

keyword: Internet-Draft
category: info
ipr: trust200902
area: Applications

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
- ins: J. Montoya
  name: Jose Montoya
  org: Mountain State Software Solutions
  email: jmontoya@ms3-inc.com

normative:
  RFC1738: # URL
  RFC3986: # JSON
  RFC6901: # JSON Pointer
  RFC7320: # URI Design and Ownership
  W3C.yaml:
    target: http://www.yaml.org/spec/1.2/spec.html
    title: YAML Aint Markup Language
    author:
    - name: Oren Ben-Kiki
      ins: Ben Kiki, O.
    - name: Clark Evans
      ins: Evans, C.
    - name: Ingy döt Net
      ins: I. Net
    date: 2009
  CommonMark:
    target: https://spec.commonmark.org/0.27/
    title: CommonMark Spec
    author:
      name: John McFarlane

informative:
  RFC8288: # Web Linking
  REST:
    target: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
    title: Architectural Styles and the Design of Network-based Software Architectures
    author:
      ins: R. Fielding
      name: Roy Thomas Fielding
      org: University of California, Irvine
    date: 2000
    seriesinfo:
      "Ph.D.": "Dissertation, University of California, Irvine"
    format:
      PDF: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
  untangled:
    target: http://roy.gbiv.com/untangled/
    title: Untangled, musings of Roy T. Fielding
    author:
      ins: R. Fielding
      name: Roy Thomas Fielding
  yahoo.rest:
    target: https://groups.yahoo.com/neo/groups/rest-discuss/info
    title: The REST Architectural Style List
  OAS:
    title: OpenAPI Specification
    target: https://github.com/OAI/OpenAPI-Specification/blob/v3.1.0-dev/versions/3.1.0.md
    author:
      org: OpenAPI Initiative, a Linux Foundation Collaborative Project

--- abstract

This document defines XREL, a data format for describing extended hypermedia relationships identified by Uniform Resource Locators.

--- middle

# Introduction

This document defines XREL, a data format for describing extended hypermedia relationships identified by Uniform Resource Locators.

This document registers a media-type identifier with the IANA: `application/xrel`. This registration is for community review and will be submitted to the IESG for review, approval, and registration with IANA.

## Definitions and Terminology

### Terminology and Conformance Language

{::boilerplate bcp14}

### General

Representational State Transfer, or **REST**, is an architectural style for distributed hypermedia systems. Introduced and first defined in 2000 in Chapter 5, REST, of the doctoral dissertation "Architectural Styles and the Design of Network-based Software Architecture" by Roy Fielding.

**Hypermedia**, or hypertext, is defined by the presence of application control information embedded within, or as a layer above, the presentation of information. Hypermedia allows for a virtually unbound network of resources while also guiding users through an application as they navigate said relationships.

A **hypermedia relationship**, also known as a link relation, describes the semantics behind a virtual uni-directional association between two resources.

A **hypermedia relationship name** is an identifier for a hypermedia relationship.

A **resource** is the intended conceptual target of a hypertext reference.

**Representational state** indicates the current state of the requested resource, the desired state for the requested resource, or the value of some other resource, such as a representation of the input data within a client’s query form, or a representation of some error condition for a response.

**Application state** is the state of the user’s application of computing to a given task, controlled and stored by the user agent and can be composed of representations from multiple servers.

This document and the specification documented in it are heavily influenced by the [OpenAPI 3.1 Spec](#OAS).

### Documents description

## Motivation

The Uniform Interface constraint of the REST architectural style dictates that hypermedia be the engine of application state. This means that the state of the application and its potential transitions are dictated by the presence of hypermedia relationships in-band and by the navigation of those relationships by an user (human or automated). In order for users to evaluate and select the appropriate relationships to navigate they must rely on an out-band understanding of relationships by their names.

While humans can derive meaning from relationship names in natural language, automated agents have relied on a central repository of standard names maintained by the Internet Assigned Numbers Authority (IANA). Instead of creating and registering entirely new link relations (i.e. `patient`, `appointment`, `schedulingService`, etc.) with a central repository, authors can choose to create an XREL document; one that explains the vital, perhaps domain-specific, semantics of the relationship and which is identified by an URL controlled by the author.

This decentralization allows for a much lower entry barrier, which is not inconsistent with the general concept of the web, and enables different use cases. For example, a private organization is fully capable of defining their own repository of XREL definitions outside of the open Internet, after all standards are a byproduct of authority. Conversely, public XREL definitions would allow for serendipitous reuse, where useful relationships backed by stable URLs might be discovered and possibly become de facto standard.

# Specification

## Format
Following OpenAPI conventions, XREL documents are YAML documents. The file extension `.yaml` SHALL be used to designate files containing XREL documents.

This specification uses [YAML 1.2](#W3C.yaml) as its underlying format. YAML is a human-readable data format that aligns well with the design goals of this specification. As in YAML, all nodes such as keys, values, and tags, are case-sensitive.

## Rich Text Formatting
Throughout the specification `description` fields are noted as supporting CommonMark markdown formatting.

Where XREL tooling renders rich text it MUST support, at a minimum, markdown syntax as described by [CommonMark 0.27](#CommonMark). Tooling MAY choose to ignore some CommonMark features to address security concerns.

## Schema
In the following description, if a field is not explicitly **REQUIRED** or described with a MUST or SHALL, it can be considered OPTIONAL.

### Relationship Object

Explains the semantics of a hypermedia relationship.

#### Properties

Name | Type | Description
---|:---:|---
description | string | **REQUIRED** Describes the semantics of a hypermedia relationship. The semantics SHOULD describe the relationship of the target resource to the context resource, and not any particular representation formats. Markdown MAY be used for rich text representation.

## Document Types

### XREL Document

Type | Description
---|---
[Relationship Object](#relationship-object) | A single Relationship Object.

#### Example

~~~ yaml
description: Refers to an event scheduling service resource related to the context resource.
~~~

### XREL Collection Document

Type | Description
---|---
Map[`string`, [Relationship Object](#relationship-object)] | A map where the keys are the document scoped relationship names and the values are Relationship Objects. XREL Collections can be used to group any number of Relationship Objects.

#### Example

~~~ yaml
scheduling-service:
  description: Refers to an event scheduling service resource related to the context resource.
patient:
  description: Refers to a patient resource related to the context resource.
~~~

## Identifying XREL Documents

XREL documents are identified by unique URLs, these URL SHOULD be dereferenceable.

In order to reduce load on servers responding to XREL document requests, it is RECOMMENDED that servers use cache control directives that instruct client apps to locally cache the results. Clients making these XREL document requests SHOULD honor the server's caching directives.

## Fragment identifiers

When applied to an XREL document, a URI fragment identifier MUST be a [JSON Pointer](RFC6901) and be computed as such.

## Identifying XREL Relationships

In the case of XREL Documents as specified in Section 2.3, the URL that identifies that document also identifies the hypermedia relationship described in that document. For example, if the document example in Section 2.3.1 is served at **http://docs.example.org/xrels/sheduling-service** then this URL is the identifier for the relationship described in that document.

In the case of XREL Collection Documents as specified in Section 2.4, fragment identifiers MUST be used for the relationships objects described in that document. For example, if the document example in Section 2.4.1 is served at **http://docs.example.org/xrels/clinical** then **http://docs.example.org/xrels/clinical#/sheduling-service** and **http://docs.example.org/xrels/clinical#/patient** identify the first and second Relationship Objects, respectively.

# IANA Considerations

This specification establishes the media type `application/xrel` for community review and will be submitted to the IESG for review, approval, and registration with IANA.

## application/xrel

**Type name:** application

**Subtype name:** xrel

**Required parameters:** none

**Optional parameters:**

  > **type**: The "type" parameter has a value of "collection" or "single".

  > Neither the parameter name nor its value are case sensitive.

  > The value "single" indicates that the media type identifies an XREL Document.

  > The value "collection" indicates that the media type identifies an XREL Collection Document.

  > If not specified, the type is assumed to be "single".

**Encoding considerations:**

  > **binary:**  Because of YAML's relation to JSON the same encoding considerations of application/json as specified in [JavaScript Object Notation](#RFC3986) apply.

**Security considerations:**

  > Because of YAML's relation to JSON this format shares security issues common to all JSON content types. The security issues of [JavaScript Object Notation](#RFC3986), section 6, should be considered.

**Interoperability considerations:** none

**Fragment identifier considerations:**

  > Fragment identifiers are to be computed as defined by the [JSON Pointer](RFC6901) specification.

**Published specification:** This Document

**Applications that use this media type:** Various

**Additional information:**

  > **magic number(s):** none

  > **file extensions:** .yaml

  > **macintosh type file code:** TEXT

  > **object idenfiers:** none

**Person to contact for further information:**

  > **Name:** Jose Montoya

  > **Email:** jmontoya@ms3-inc.com

**Intended usage:** Common

**Author/change controller:** Jose Montoya

# Appendix A. Acknowledgments

Many thanks to Mike Amundsen, Jeff Michaud, Stu Charlton, Eric Wilde and Darrel Miller for their contributions in this space, even if not directly related to XREL documents.

# Appendix B. Frequently Asked Questions

## How can I submit comments or feedback to the editors?

The issues list for this draft can be found at <https://github.com/phtal-org/internet-draft-xrel/issues>. For additional information, see <https://phtal-org.github.io/internet-draft-xrel/>.

To provide feedback, use this issue tracker, the communication methods listed on the homepage, or email the document editors.

## Why not include target attributes as defined by RFC8288 'Web Linking'?
Link relations are universal, they describe an *association* to a conceptual target and not the targets themselves nor their representations. It is the responsibility of the application authors to communicate to their clients what data types are necessary to navigate a relationship and/or the data types that might be expected as a result.

This level of abstraction has value because it's easier to standardize representations (HTML Microformats, RAML data types, etc.) and link relations than it is to standardize objects and object-specific interfaces. Application servers are free to combine representations and link relations in any way they wish and to provide them in any order, all while remaining understandable to the client.
