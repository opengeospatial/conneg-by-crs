|Internet Engineering Task Force|J. Abhayaratna|
|-----|-----|
|Internet-Draft|Open Geospatial Consortium|
|Intended status|Standards Track|
|Expires|December 30, 2019

# Negotiating Content by Coordinate Reference System (CRS)
draft-abhayaratna-accept-crs-00

## Abstract
This document defines two new HTTP headers that enable User Agents and hosts to indicate and negotiate the coordinate reference system (CRS) used to locate spatial information returned by specific resource. In this context, a coordinate reference system is a document describing how to interpret spatial information in addition to the semantic integration provided by a [PROFILE] and the syntactical interpretation provided by a MIME type. Examples of coordinate reference systems include World Geodetic System 1984 (WGS84) and the Geodetic Datum of Australia 1994 (GDA94).

DO WE HAVE OR NEED IANA LINK HEADERS?
Further, it defines and registers the __"crs"__ parameter for the HTTP Link header and suggests a best practice for the use of the new headers together with the Link header to perform content negotiation and point clients to alternate coordinate reference systems.

## Status of this Memo
This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79.

Internet-Drafts are working documents of the Internet Engineering Task Force (IETF). Note that other groups may also distribute working documents as Internet-Drafts. The list of current Internet-Drafts is at https://datatracker.ietf.org/drafts/current/.

Internet-Drafts are draft documents valid for a maximum of six months and may be updated, replaced, or obsoleted by other documents at any time. It is inappropriate to use Internet-Drafts as reference material or to cite them other than as "work in progress."

This Internet-Draft will expire on December 30, 2019.

## Copyright Notice
Copyright (c) 2019 IETF Trust and the persons identified as the document authors. All rights reserved.

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents (https://trustee.ietf.org/license-info) in effect on the date of publication of this document. Please review these documents carefully, as they describe your rights and restrictions with respect to this document. Code Components extracted from this document must include Simplified BSD License text as described in Section 4.e of the Trust Legal Provisions and are provided without warranty as described in the Simplified BSD License.

## Table of Contents

## 1. Introduction

This document defines two new HTTP headers that enable User Agents (UAs) and hosts to indicate and negotiate the coordinate reference system (CRS) used to locate a specific resource. On the Web, resources are identified with HTTP(S) URIs. In many cases when the resource includes spatial information, it can be desired to have multiple coordinate reference systems of a resource to accommodate different UAs or use cases. Indeed, this requirement is covered by the W3C Spatial Data on the Web (SDW) best practices, specifically [best practice 7](https://www.w3.org/TR/sdw-bp/#bp-crs-choice), specifically the statement :

_"When publishing spatial data, it is best to help users avoid the need for them to transform spatial data between coordinate reference systems themselves by providing data in a form, or forms, which they can use directly. "_

When an Agent issues a GET request for a specific URI, the Agent and the server negotiate which of the available coordinate reference systems best suit the Agent's needs and capabilities. Typically a Agent specifies preferences by setting appropriate HTTP headers, e. g. Accept for media type, Accept-Language for content language or Accept-Charset for character encoding.

Identification of coordinate reference system has been handled in a variety of ways in the past. It is generally done by identifying some attribute (e.g., crs, datum, etc.,) that will be used to pass it backwards and forwards, and some vocabulary for the value being passed (e.g., WGS84, EPSG:4326, etc.,). These have tended to vary between implementations as indicated, causing clients to have to handle interactions with different resources independently. This document proposes a solution by defining the HTTP headers Accept-Crs and Content-Crs.

### 1.1. Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 2. Terminology and Implementation Options

### 2.1. A Note on Terminology

In the context of this proposal, a "coordinate reference system" "...is a coordinate-based local, regional or global system used to locate geographical entities" [(https://en.wikipedia.org/wiki/Spatial_reference_system)](https://en.wikipedia.org/wiki/Spatial_reference_system). Examples of a "coordinate reference system" in this context include, but are not limited to, WGS 84 and GDA 94. How those coordinate reference systems are used by consuming applications is beyond the scope of this proposal, but typical use cases are interpretation of a spatial feature in order to render it and transform between one coordinate reference system and another.

### 2.2. Other Implementation Options Considered

A number of options were considered when specifying how coordinate reference system negotiation could be implemented. These were identified based on existing implementations which help to argue the value of standardization. Below is a record of those options listing the consequences of using them:

#### 2.2.1 Pass in body

Currently, many GeoJSON API based implementations support communication of CRS. The GeoJSON standard does not provide standardized method of negotiating CRS. As a result, each implementation varies with respect to both how CRS is requested, as well as the method used to communicate which CRS was used to locate the data returned. Most, if not all, implementations use the body of the request or response for communication.

There are three issues with this implementation option. Firstly, the name of the parameters used in requests and responses vary between implementations. For example, the variation of terms used to specify the name of the parameter include, but are not limited to, "SRS" (the acronym for Spatial Reference System) and "CRS". Secondly, the semantics of the value used in requests and responses also varies between implementations. For example, the variation of terms used to specify the name of the value for the World Geodetic System of 1984 include, but are not limited to, "WGS84", "4326", and "EPSG:4326". Finally, the value returned by a service does not necessary support the ability to dereference the CRS.

#### 2.2.2 Use headers but reject when Accept-Crs is not supported

Given the issues associated with option 1 above, some services have standardized on the use of HTTP headers, and the names of these headers (Accept-Crs and Content-Crs), to communicate the both the CRS required by the UA and the CRS used to locate the spatial information returned. Whilst this solves the first of the three problems identified above, it does not solve the remaining two.

There is a level of standardization relating to the what values are passed as well, usually referencing the International Association of Oil and Gas Providers' (IOGP's) European Petroleum Survey Group (EPSG) codes. When a service makes a request using an Accept-Crs header containing an EPSG code that is not supported by the implementation, a [406 HTTP Code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) (Not Acceptable) is returned by the service. There are two issues with this implementation option. Firstly, whilst this implementation is being adopted, many implementations will not use the headers required by it. When this happens, either a default Content-Crs will be returned, or the implementation will return the error. The first option is preferrable as the second approach limits uptake as implementers of services are unlikely to want to reject requests from Agents that aren't aware of the headers. Secondly, this implementation option does not enable a user to dereference the CRS.

## 3. Client and Server Behaviour

The "Accept-Crs" and "Content-Crs" header fields can be sent by both the UA and the server. The "Accept-Crs" header is used to specify one or more CRSs the Agent can accept, whereas the "Content-Crs" header tells the Agent which CRS is used to locate the spatial features in the payload of the message. Following this approach, an Agent issuing a request for a resource can specify that it prefers spatial information to be located using the CRS defined by EPSG code 4283, but that the CRS defined by EPSG code 4326 is also acceptable by setting the "Accept-Crs" header field appropriately. When the server answers, it would set the "Content-Crs" header field that describes the CRS used to locate the spatial features it returns. Likewise, an Agent sending spatial data to a server would set the the "Content-Crs" header fields. If the server cannot process the specified profile, it would answer with an http 406 status code and possibly a list of acceptable CRSs.

An "Accept-Crs" and "Content-Crs" header field do not contain the actual definition of the CRS but instead points to it using a URI. As long as the URI is only used to denote the CRS, the URI does not need to point to an actual document but can be considered opaque. If the parties involved agree on a CRS definition, the CRS can be identified with e.g. a URN or an info-URI. When a protocol-based URI, such as an FTP- or an HTTP-URI is used, however, it is RECOMMENDED that it dereference to a document containing the CRS definition.

The [Open Geospatial Consortium (OGC)](http://www.opengeospatial.org) provides a definitions server that enables Agents and servers to dereference CRSs. [NOTE this is still somewhat limited in terms of formats available for CRS definition but will be updated to meet practical needs of this proposal] Through negotiation between CRS registers such as the EPSG registry, this service provides a method of unambiguously identifying the values for CRSs. For example, the CRS defined by EPSG code 4326 can be unambiguously identified by the following URI: [http://www.opengis.net/def/crs/EPSG/0/4326](http://www.opengis.net/def/crs/EPSG/0/4326). It is thus RECOMMENDED that Agents and servers use the URIs hosted by this federated register to identify CRSs in their headers. 

### 3.1. Accept-Crs Header Syntax

The "Accept-Crs" header is used to specify one or more CRSs the issuing Agent can accept for processing. Each CRS is identified by a URI reference, preferrably using a URI hosted by the OGC definitions server's federated CRS registry. If several CRSs are specified, quality values as defined in Section 5.3.1 of RFC 7230 can be used to assign a relative "weight" to the preference. Exactly how that weight is used to determine the best representation is beyond the scope of this specification.

A request without any "Accept-Crs" header field implies that the Agent will accept spatial information located by an CRS. If the header field is present in a request and the server cannot respond with data located using the specified CRS, it can either honour the header field by sending a 406 (Not Acceptable) HTTP Code response or disregard the header field by treating the response as if it is not subject to content negotiation. If the the server chooses to disregard the header field and the CRS used to locate spatial information returned by it is known, the origin server SHOULD send a "Content-Crs" header indicating that CRS together with the payload.

Figure 5 describes the syntax (Augmented Backus-Naur Form) of the header fields, using the grammar defined in RFC 5234 and the rules defined in Section 3.2 of RFC 7230. The definitions of "URI-reference" and "weight" are imported from RFC 7230 and RFC 7231, respectively.

Accept-CRS header syntax

```

Accept-CRS = "Accept-CRS" ":" (CRS-value) *("," CRS-value)
CRS-value = "<" URI-reference ">"

```
Figure 5

### 3.2. Content-Crs Header Syntax

The "Content-Crs" header field is used to specify a CRS used to locate spatial information in the payload. The CRS is identified by a URI reference, preferrably using a URI hosted by the OGC definitions server's federated CRS registry. If a client uses the "Accept-Crs" header to specify one or more CRSs it is willing to accept and a server does not use the "Content-Crs" header to specify which CRS was used to locate spatial information it returns, the Agent MAY process the returned content as it deems fit.

If an Agent uses the "Content-Crs" header field to indicate the CRS used to locate the payload it sends (e. g. in an HTTP POST or PUT request) and the server cannot process content located using that CRS, the server SHOULD send a 406 (Not acceptable) HTTP Code response together with an "Accept-Crs" header (including q-values) to indicate the CRSs it can process. Reasons for not sending a 406 HTTP Code response in such a case might be that the the processing of the message payload leads to an internal server error. If in such a case the server does not implement content negotiation by CRS, the server is more likely to return a 500 HTTP Code (Internal server error) response instead of 406 HTTP Code response.

Figure 6 describes the syntax (Augmented Backus-Naur Form) of the header fields, using the grammar defined in RFC 5234 and the rules defined in Section 3.2 of RFC 7230. The definition of "URI-reference" is imported from RFC 7230.

CRS header syntax

```

CRS = "Content-Crs" ":" "<" URI-reference ">"

```
Figure 6

## 4. Examples

The following examples highlight the exchange of CRS information between an Agent and a server. For clarity, the examples only contain minimal information, i. e. only the relevant headers are included and message bodies are ignored.

A client requests spatial information located using the CRS identified by EPSG Code 4326. This is indicated by the URI: [http://www.opengis.net/def/crs/EPSG/0/4326](http://www.opengis.net/def/crs/EPSG/0/4326).

```

Request:
GET /some-resource HTTP/1.1
Accept-Crs: <http://www.opengis.net/def/crs/EPSG/0/4326>

Response:
HTTP/1.1 200 OK
Content-Crs: <http://www.opengis.net/def/crs/EPSG/0/4326>
Link: 	 rel="self";
		type="application/xml";
		crs="<http://www.opengis.net/def/crs/EPSG/0/4326>",
		rel="alternate";
		type="application/xml";
		crs="<http://www.opengis.net/def/crs/EPSG/0/4283>"

```
Figure 7

A client requests an RDF/XML document located using the CRS identified by EPSG code 4326. It uses q-values to express a preference for that CRS, the server, however, prefers to deliver spatial information located using the EPSG code 4283.

```

Request:
GET /some-resource HTTP/1.1
Accept: application/rdf+xml
Accept-Crs: <http://www.opengis.net/def/crs/EPSG/0/4326>,
               <http://www.opengis.net/def/crs/EPSG/0/4283>

Response:
HTTP/1.1 200 OK
Content-Type: application/rdf+xml
Content-Crs: <http://www.opengis.net/def/crs/EPSG/0/4283>
Link:	 rel="self";
	 type="application/rdf+xml",
	 rel="alternate";
	 type="text/ttl"

```
Figure 8

A client PUTs a GeoJSON document located using EPSG Code 4283. The server answers that it can only process spatial information located using EPSG Code 4326.

```

Request:
PUT /some-resource HTTP/1.1
Content-Type: application/geo+json
Content-Crs: <http://www.opengis.net/def/crs/EPSG/0/4283>

Response:
HTTP/1.1 406 Not acceptable
Content-Type: application/geo+json
Accept-Crs: <http://www.opengis.net/def/crs/EPSG/0/4326>

```
Figure 9

A client requests an XML document where the elements in namespace urn:example:namespaces:ns1 must be located using the CRS identified by EPSG Code 4326 and the elements in namespace urn:example:namespaces:ns2 must be located using the CRS identified by EPSG Code 4283. The server answers that it can supply the document as requested.

```

Request:
GET /some-resource HTTP/1.1
Accept-Crs:    <urn:example:namespaces:ns1
					http://www.opengis.net/def/crs/EPSG/0/4326
								urn:example:namespaces:ns2
											http://www.opengis.net/def/crs/EPSG/0/4283>

Response:
HTTP/1.1 200 OK
Content-Type: application/xml
Content-Crs: <urn:example:namespaces:ns1
		http://www.opengis.net/def/crs/EPSG/0/4326
			urn:example:namespaces:ns2
				http://www.opengis.net/def/crs/EPSG/0/4283>

```
Figure 10

## 5. Acknowledgements

This is the play to have YOUR NAME prominently display!
# Joseph Abhayaratna
# Lars Svenssen
# Rob Atkinson
# Ruben Verborgh

## 6. IANA Considerations

This specification defines two header fields for the Hypertext Transfer Protocol (HTTP) that have been registered with the Internet Assigned Numbers Authority (IANA) following the "Registration Procedures for Message Header Fields" RFC 3864. [TO BE REMOVED: This registration should take place at the following location: http://www.iana.org/assignments/message-headers/message-headers.xhtml] Further, it defines a parameter for the "Link" header. There is no registry for link-param except for the values listed in [RFC5988] so the parameter "crs" will be seen as a link-extension.

### 6.1. Accept-Crs HTTP Header Registration

The Accept-Crs header should be added to the permanent registry of message header fields (see RFC 3864), taking into account the guidelines given by HTTP/1.1 RFC 7231".

Header Field Name: Accept-Crs

Applicable Protocol: Hypertext Transfer Protocol (HTTP)

Status: Standard

Author/Change controller: IETF

Specification document(s): RFC XXXX

### 6.2. Content-Crs HTTP Header Registration

The Content-Crs header should be added to the permanent registry of message header fields (see RFC 3864), taking into account the guidelines given by HTTP/1.1 RFC 7231".

Header Field Name: Content-Crs

Applicable Protocol: Hypertext Transfer Protocol (HTTP)

Status: Standard

Author/Change controller: IETF

Specification document(s): RFC XXXX

## 7. Security Considerations

The Accept-Crs and Content-Crs headers may expose information that a User Agent or an origin server would prefer to not publish. In such a case, a server can simply stop exposing the header, in which case HTTP interactions would be back to the level of standard HTTP (i.e., with no indication what CRS the Agent or server prefer and/or can handle.

## 8. References

### 8.1. Normative References


[RFC2119]	  Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997.  
[RFC5234]	  Crocker, D. and P. Overell, "Augmented BNF for Syntax Specifications: ABNF", STD 68, RFC 5234, DOI 10.17487/RFC5234, January 2008.  
[RFC7230]	  Fielding, R. and J. Reschke, "Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing", RFC 7230, DOI 10.17487/RFC7230, June 2014.  
[RFC7231]	  Fielding, R. and J. Reschke, "Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content", RFC 7231, DOI 10.17487/RFC7231, June 2014.  

### 8.2. Informative References

[DSP]	Nilsson, M., "Description Set Profiles: A constraint language for Dublin Core Application Profiles", 2008.  
[IANAMEDIAREG]	 IANA, "Media Types", 2015.  
[JSONLD]	 Sporny, M., Kellogg, G. and M. Lanthaler, "JSON-LD 1.0: A JSON-based Serialization for Linked Data", 2015.  
[RFC3236]	 Baker, M. and P. Stark, "The 'application/xhtml+xml' Media Type", RFC 3236, DOI 10.17487/RFC3236, January 2002.  
[RFC3864]	 Klyne, G., Nottingham, M. and J. Mogul, "Registration Procedures for Message Header Fields", BCP 90, RFC 3864, DOI 10.17487/RFC3864, September 2004.  
[RFC3870]	 Swartz, A., "application/rdf+xml Media Type Registration", RFC 3870, DOI 10.17487/RFC3870, September 2004.  
[RFC5988]	 Nottingham, M., "Web Linking", RFC 5988, DOI 10.17487/RFC5988, October 2010.  
[RFC7240]	 Snell, J., "Prefer Header for HTTP", RFC 7240, DOI 10.17487/RFC7240, June 2014.  
[SHACL]		 Knublauch, H. and A. Ryman, "Shapes Constraint Language (SHACL): W3C First Public Working Draft 08 October 2015", 2014.  
[TURTLE]	 Prud'hommeaux, E. and G. Carothers, "RDF 1.1 Turtle", 2014.  
[XMLBEANS]	 Apache Software Foundation, "XMLBeans", 2014.  
[XMLSCHEMA]	 Gao, S., Sperberg-McQueen, C. and H. Thompson, "W3C XML Schema Definition Language (XSD) 1.1 Part 1: Structures", 2012.  
[XSD]		 Thompson, H., Beech, D., Maloney, M. and N. Mendelsohn, "XML Schema Part 1: Structures Second Edition", 2004.  
[PROFILE]	 Svensson, L. and Verborgh, R., "Negotiating Profiles in HTTP", 2017.  

## Authors' Addresses

Joseph Abhayaratna  
PSMA Australia  
Unit 6 Level 2  
113 Canberra Avenue  
Griffith ACT 2603  
Email: Joseph.Abhayaratna@psma.com.au  

Rob Atkinson  
