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
  github.md:
    target: https://github.github.com/gfm/
    title: GitHub Flavored Markdown Spec
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
  RAML:
    title: RAML Specification
    target: https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md
    author:
      org: The RAML Workgroup

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

**Hypermedia** is defined by the presence of application control information embedded within, or as a layer above, the presentation of information.

A **hypermedia relationship**, also known as a link relation, describes the semantics behind a virtual uni-directional association between two resources. Hypermedia relationships allow for a virtually unbound network of resources while also guiding users through an application as they navigate said relationships.

A **hypermedia relationship name** is an identifier for a hypermedia relationship.

A **resource** is the intended conceptual target of a hypertext reference.

**Representational state** indicates the current state of the requested resource, the desired state for the requested resource, or the value of some other resource, such as a representation of the input data within a client’s query form, or a representation of some error condition for a response.

**Application state** is the state of the user’s application of computing to a given task, controlled and stored by the user agent and can be composed of representations from multiple servers.

This document and the specification documented in it are heavily influenced by the [RAML 1.0 Spec](#RAML).

### Documents description

A trailing question mark, for example **description?**, indicates an optional property.

Throughout this specification, **markdown** means [GitHub-Flavored Markdown](#github.md). Tooling MAY choose to ignore some Markdown features to address security concerns.

## Motivation

The Uniform Interface constraint of the REST architectural style dictates that hypermedia be the engine of application state. This means that the state of the application and its potential transitions are dictated by the presence of hypermedia relationships in-band and the navigation of those relationships by an user (human or automated). In order for users to evaluate and select the appropriate relationships to navigate they must rely on an out-band understanding of relationships by their names.

While humans can derive meaning from relationship names in natural language, automated agents have relied on a central repository of de jure standard names maintained by the Internet Assigned Numbers Authority (IANA). Instead of creating and registering entirely new link relations (i.e. `patient`, `appointment`, `schedulingService`, etc.) with a central repository, authors can create an XREL document; one that explains the vital, perhaps domain-specific, semantics of the relationship and which is identified by an URL controlled by the author.

This decentralization allows for a much lower entry barrier, which is not inconsistent with the general concept of the web, and enables different use cases. For example, a private organization is fully capable of defining their own repository of XREL definitions outside of the open Internet, after all standards are a byproduct of authority. Conversely, public XREL definitions would allow for serendipitous reuse, where useful relationships backed by stable URLs might be discovered and possibly become de facto standard.

# XREL Documents

## Format

Following RAML conventions, XREL documents are YAML documents. The file extension `.yaml` SHALL be used to designate files containing XREL documents.

This specification uses [YAML 1.2](#W3C.yaml) as its underlying format. YAML is a human-readable data format that aligns well with the design goals of this specification. As in YAML, all nodes such as keys, values, and tags, are case-sensitive.

To facilitate the automated processing of XREL documents, XREL imposes the following restrictions and requirements in addition to the core YAML 1.2 specification:

* The first line of a XREL file consists of a YAML comment that specifies the XREL version and document type. Therefore, RAML processors cannot completely ignore all YAML comments.

## Relationship Object

Explains the semantics of a hypermedia relationship.

### Properties

Name | Type | Description
---|:---:|---
description | string | Describes the semantics of a hypermedia relationship. The semantics SHOULD describe the relationship of the target resource to the context resource, and not any particular representation formats. Markdown MAY be used for rich text representation.

## XREL Document

Defines a single Relationship Object.

The first line of a XREL document MUST begin with the text `#%XREL 1.0` followed by nothing but the end of the line.

### Document Example

~~~ yaml
#%XREL 1.0

description: Refers to an event scheduling service resource related to the context resource.
~~~

## XREL Collection Document

Defines a map where the keys are the document scoped relationship names and the values are Relationship Objects. XREL Collections can be used to combine any collection of Relationship Objects.

The first line of a collection document MUST begin with the text `#%XREL 1.0 Collection` followed by nothing but the end of the line.

### Document Example

~~~ yaml
#%XREL 1.0 Collection

schedulingService:
  description: Refers to an event scheduling service resource related to the context resource.
patient:
  description: Refers to a patient resource related to the context resource.
~~~

## Identifying XREL Documents

XREL documents are identified by unique URLs, this URL SHOULD be dereferenceable.

In order to reduce load on servers responding to XREL document requests, it is RECOMMENDED that servers use cache control directives that instruct client apps to locally cache the results. Clients making these XREL document requests SHOULD honor the server's caching directives.

## Fragment identifiers

When applied to an XREL document, a URI fragment identifier MUST be a [JSON Pointer](RFC6901) and be computed as such.

## Identifying XREL Relationships

In the case of XREL Documents as specified in Section 2.3, this URL identifies the hypermedia relationship described in that document. For example, if the document example in Section 2.3.1 is served at **http://docs.example.org/xrels/shedulingService** then this URL is the identifier for the relationship described in that document.

In the case of XREL Collection Documents as specified in Section 2.4, fragment identifiers MUST be used for the relationships objects described in that document. For example, if the document example in Section 2.4.1 is served at **http://docs.example.org/xrels/clinical** then **http://docs.example.org/xrels/clinical#/shedulingService** and **http://docs.example.org/xrels/clinical#/patient** identify the first and second Relationship Object, respectively.

# IANA Considerations

This specification establishes the media type `application/xrel` for community review and will be submitted to the IESG for review, approval, and registration with IANA.

## application/xrel

**Type name:** application

**Subtype name:** xrel

**Required parameters:** none

**Optional parameters:**

  > **profile:**: A whitespace-separated list of URIs identifying specific constraints or conventions that apply to an XREL document. A profile must not change the semantics of the resource representation when processed without profile knowledge, so that clients both with and without knowledge of a profiled resource can safely use the same representation. The profile parameter may also be used by clients to express their preferences in the content negotiation process. It is recommended that profile URIs are dereferenceable and provide useful documentation at that URI.

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

Many thanks to Mike Amundsen, Jeff Michaud, Eric Wilde and Darrel Miller for their contributions in this space, even if not directly related to XREL documents.

# Appendix B. Frequently Asked Questions

## How can I submit comments or feedback to the editors?

The issues list for this draft can be found at <https://github.com/phtal-org/internet-draft-xrel/issues>. For additional information, see <https://phtal-org.github.io/internet-draft-xrel/>.

To provide feedback, use this issue tracker, the communication methods listed on the homepage, or email the document editors.

## Why not include target attributes as defined by RFC8288 'Web Linking'?
Link relations are universal, they describe an *association* to a conceptual target and not the targets themselves nor their representations. It is the responsibility of the application authors to communicate to their clients what data types are necessary to navigate a relationship and/or the data types that might be expected as a result, also known as typed link relations.

This level of abstraction has value because it's easier to standardize representations (HTML Microformats, RAML data types, etc.) and link relations than it is to standardize objects and object-specific interfaces. Application servers are free to typify link relations in any way they wish and to provide them in any order, all while remaining understandable to the client.
