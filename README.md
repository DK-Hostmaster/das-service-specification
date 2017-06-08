# DK Hostmaster Domain Availability Service Specification

2016/10/18
Revision: 1.6

# Table of Contents

<!-- MarkdownTOC bracket=round depth=3 -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
	- [License](#license)
	- [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Domain Availability Service](#domain-availability-service)
- [Available Environments](#available-environments)
	- [Production Environment](#production-environment)
	- [Sandbox Environment](#sandbox-environment)
- [Implementation Limitations](#implementation-limitations)
	- [AAA](#aaa)
	- [Supported Media-types](#supported-media-types)
	- [Rate Limiting](#rate-limiting)
	- [Waiting List](#waiting-list)
- [Session Handling](#session-handling)
- [Domain Status](#domain-status)
		- [Available: `available`](#available-available)
		- [Unavailable: `unavailable`](#unavailable-unavailable)
		- [Blocked: `blocked`](#blocked-blocked)
		- [Available for designated user from waiting list: `available-on-waiting-list`](#available-for-designated-user-from-waiting-list-available-on-waiting-list)
- [Service `/domain/is_available`](#service-domainisavailable)
	- [Request](#request)
	- [Examples for unavailable domain](#examples-for-unavailable-domain)
	- [Examples for available domain](#examples-for-available-domain)
	- [Example with bad domain parameter](#example-with-bad-domain-parameter)
	- [Example with bad credentials](#example-with-bad-credentials)
- [Test Data](#test-data)
	- [Domains](#domains)
	- [Accounts / Credentials](#accounts--credentials)
- [References](#references)
- [Resources](#resources)
	- [Mailing list](#mailing-list)
	- [Issue Reporting](#issue-reporting)
	- [Additional Information](#additional-information)
	- [Demo Client](#demo-client)
- [Appendices](#appendices)
	- [HTTP Status Codes](#http-status-codes)

<!-- /MarkdownTOC -->

<a name="introduction"></a>
# Introduction

This document describes and specifies the implementation offered by DK Hostmaster A/S for interaction with the central registry for the ccTLD dk using the Domain Availability Service (DAS). It is primarily aimed at a technical audience, and the reader is required to have prior knowledge of HTTP and possibly DNS registration.

<a name="about-this-document"></a>
# About this Document

This specification describes version 1 (1.0.x) of the DK Hostmaster DAS Implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster DAS implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/das-service-specification/blob/master/README.md), using the gitprint service.

<a name="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a name="document-history"></a>
## Document History

* 1.6 2016-10-18
  * Added information on new status `available-on-waiting-list`

* 1.5 2016-09-01
  * Minor clarification on credentials 

* 1.4 2016-06-09
  * Removed obsolete datasheet

* 1.3 2016-06-09
  * Added link to demo client, also available on Github  

* 1.2 2016-04-19
  * Filled in data in the datasheet, more information will follow
  * Filled in details on blocking policy for failed login attempts based on user-id and IP-address 
  * Added link to the gitprint service

* 1.1 2015-09-02
  * Migrated to markdown and hosting on Github, no changes to actual content just formatting 

* 1.0 2013-02-25
  * Initial revision

<a name="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.
The service is not subject to any sorts of standards, but adheres to practices in implementation of REST and use of HTTP in context of REST.

<a name="domain-availability-service"></a>
# Domain Availability Service

The DK Hostmaster’s DAS is based on a SOA architecture. The implementation is regarded as a service offered to external parties requiring inquiry actions towards the DK Hostmaster registry.

DAS is an HTTP-based protocol aimed at providing a speedy interface for requesting information from the DK Hostmaster registry. The service is intended for machine-to-machine communication in a client-server setup. Please see the References chapter for more information on specifications and references for HTTP and related.

The service requires the use of and possible development of client software. This is beyond the scope of this specification as the API and other assets for assisting in this are the primary object of this document.

In addition to the assets, DK Hostmaster aims to assist users and developers of possible client software with integration towards DK Hostmaster and therefore provide facilities to ease this integration. This is primarily centered around a sandbox environment and related documentation.

<a name="available-environments"></a>
# Available Environments

DK Hostmaster offers the following environments:

| Environment | Role | Policies |
| ----------- | ---- | ----------- |
| production  | production | This environment will be the production environment for the DK Hostmaster Domain Availability Service |
| sandbox     | development | This environment is intended for client development towards the DK Hostmaster Domain Availability Service. |

<a name="production-environment"></a>
## Production Environment

* is_available requests made to this environment will reflect live production data
* production credentials and proper authorization are needed access the service

<a name="sandbox-environment"></a>
## Sandbox Environment

* is_available requests made to this environment will reflect dummy data. 
* Please see the section on test data. 
* The sandbox does not implement actual rate limiting, but offers simulated rate limiting by using a specific request, please see the section on test data.

<a name="implementation-limitations"></a>
# Implementation Limitations

In general the service is not localized and all DAS related errors and messages are provided in English. 

The only localization support provided by the service is the support for IDN domains. Please note the service require requests in UTF-8, meaning punycode encoded domain names will be interpreted as-is, meaning in ASCII context. 

The punycode encoded example of: xn--kdplg-orai3l.dk will be evaluated as xn--kdplg-orai3l.dk and not decoded to kødpålæg.dk prior to evaluation.

<a name="aaa"></a>
## AAA

This service is called using Basic HTTP Authentication supporting current authentication credentials, consisting of:

- user-id / handle
- password

Too many failed login attempts will block the account. The block for a user-id lasts for 24 hours and it automatically lifted. 

If failing login attempts continue or is spread across user-ids originating from the same IP-address the IP-address will be blocked. The block for an IP-address lasts for 24 hours and it automatically lifted. 

<a name="supported-media-types"></a>
## Supported Media-types

The service supports JSON, XML and plain text format, using the UTF-8 character set. In order to specify what format you want to retrieve the format should be specified in the HTTP header: Accept-header.
Control the content type by setting header info, using the below examples:

- `Accept: application/json; charset=utf-8`
- `Accept: application/xml; charset=utf-8`
- `Accept: text/plain; charset=utf-8`

If content type is not specified, response will reflect this with an HTTP status code: 415 (see: HTTP status code listing in appendices).

<a name="rate-limiting"></a>
## Rate Limiting

We only allow a certain number of requests per minute. We reserve the right to adjust the rate limit in order to provide a high quality of service. 
If rate limit is exceeded the HTTP status code 429 “Too many requests” is returned. Further, the response will have a `Retry-After` header that tells you for how many seconds to wait before retrying. 

Current limit is set to 60 requests per minute.

Please note the sandbox environment is not under rate limiting at this time, due to a wish for unlimited use for developers. 

<a name="waiting-list"></a>
## Waiting List

DK Hostmaster offers a waiting list service for domainnames, when a domainname becomes available to the first position on a waiting list, it should be registered using the standard registration proces either using the **email form** or **EPP**. This influences the create domain command, which should just be populated with the userid of the user which has been pre-approved for registration of the domainname with DK Hostmaster.

Please refer to the DK Hostmaster A/S [website](https://www.dk-hostmaster.dk/en/waiting-list) for more information.

<a name="session-handling"></a>
# Session Handling

The service uses a basic session handling based on cookies. 

| Parameter | Value | Description |
|-----------|-------|-------------|
| cookie name | dkhm-das-session | This is the name of the cookie |
| cookie domain | .dk-hostmaster.dk |  
| expiration | 3600 seconds | The expiration date provided in the cookie is in the GMT timezone |

<a name="domain-status"></a>
# Domain Status

The service returns a queried domain name and it's status if possible. The different domain status has the following meanings:

<a name="available-available"></a>
### Available: `available`

A given domain name is available for application.

<a name="unavailable-unavailable"></a>
### Unavailable: `unavailable`

A given domain name is in use and is not available for application.

<a name="blocked-blocked"></a>
### Blocked: `blocked`

A given domain name is in a special state where the application is handled by the registrant, but is available for application.

Please note that a `blocked` domain name can be interpreted as `available`, since the domain is available for application eventhought the application process is different. 

Please refer to the: [General Terms and Conditions][general_terms_and_conditions], section 3.3.2.

<a name="available-for-designated-user-from-waiting-list-available-on-waiting-list"></a>
### Available for designated user from waiting list: `available-on-waiting-list`

A given domain name has been offered to the first entry on a waiting list and is awaiting the specific user's approval or decline to the this offer.

<a name="service-domainisavailable"></a>
# Service `/domain/is_available`

<a name="request"></a>
## Request

Method: 
    `GET`

URL path: 
    `/domain/is_available/<domain>`

| Parameter | Type | Description | Mandatory | Example |
|-----------|------|-------------|-----------|---------|
| domain    | string | The domain name to evaluate, it has to adhere to the domain name format expected by DK Hostmaster, see References. | yes | abc.dk, jordbærgrød.dk |
| status | enumerated string | string indicating status of request, either one of: `available`, `unavailable`, `blocked` or `available-on-waiting-list` | yes | |
| message | enumerated string | string providing human readable message, “ok” on success | optional |

Default HTTP header observed: 200 OK. Additional status data in Status and Message. For additional HTTP status codes, which can be exhibited by the service, please refer to the addendum.

<a name="examples-for-unavailable-domain"></a>
## Examples for unavailable domain

JSON:
```Shell
% curl --header Accept:application/json \
https://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/dk-hostmaster.dk
```

```JSON
{"domain":"dk-hostmaster.dk","status":"ok","domain_status":"unavailable"}
```

XML:
```Shell
% curl --header Accept:application/xml \ 
https://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/dk-hostmaster.dk
```

```XML
<?xml version='1.0' encoding='UTF-8' standalone='yes'?>
<response>
  <domain>dk-hostmaster.dk</domain>
  <domain_status>unavailable</domain_status>
</response>
```

Text:
```Shell
% curl --header Accept:text/plain \
http://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/dk-hostmaster.dk
```

```
domain:dk-hostmaster.dk
status:ok
domain_status:unavailable
```

<a name="examples-for-available-domain"></a>
## Examples for available domain

JSON:
```
% curl --header Accept:application/json \
https://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/adsf.dk
```

```JSON
{"domain":"adsf.dk","status":"ok","domain_status":"available"}
```

XML:
```Shell
% curl --header Accept:application/xml \ 
https://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/asdf.dk
```

```XML
<?xml version='1.0' encoding='UTF-8' standalone='yes'?>
<response>
  <domain>adsf.dk</domain>
  <domain_status>available</domain_status>
  <status>ok</status>
</response>
```

Text:
```Shell
% curl --header Accept:text/plain \
http://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/asdf.dk
```

```
domain:adsf.dk
status:ok
domain_status:available
```

<a name="example-with-bad-domain-parameter"></a>
## Example with bad domain parameter

Please note the -v flag to curl and that the response has been stripped down.

Text:
```Shell
% curl -v --header Accept:text/plain \
http://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/asdf
```

```
< HTTP/1.1 415 Unsupported Media Type
< Connection: keep-alive
< Content-Type: application/json; charset=utf-8;
< Cache-Control: max-age=1, no-cache
< Date: Wed, 23 Oct 2013 11:59:30 GMT
< Content-Length: 24
< Server: Mojolicious (Perl)
< 
"Unsupported media type"
```

<a name="example-with-bad-credentials"></a>
## Example with bad credentials

Please note the -v flag to `curl` and that the response has been stripped down.

Text:
```Shell
% curl -v --header Accept:text/plain \
http://REG-123456:secret@das-sandbox.dk-hostmaster.dk/domain/is_available/asdf
```

```
< HTTP/1.1 401 Unauthorized
< Connection: keep-alive
< Content-Type: application/json; charset=utf-8;
< Cache-Control: max-age=1, no-cache
< Date: Wed, 23 Oct 2013 12:05:22 GMT
* Authentication problem. Ignoring this.
< WWW-Authenticate: Basic realm=DKH Domain Availability Service (DAS)
< Content-Length: 56
< Server: Mojolicious (Perl)
status:error
message:User authentication error
```

<a name="test-data"></a>
# Test Data

The sandbox uses a predefined set of test data. All domains not listed in the below list will be reported as `available` in the sandbox environment.

<a name="domains"></a>
## Domains

| Domain name | Status | Notes |
|-------------|--------|-------|
| dk-hostmaster.dk | `unavailable` | The domain is active |
| waiting-list.dk | `available-on-waiting-list` | The domain status is awaiting a specific registrant |
| blocked.dk | `blocked` | The domain status is blocked |
| æøåöäüé.dk | `unavailable` | This domain is active |
| * | `available` | Everything not listed above will be reported as `available` |


<a name="accounts--credentials"></a>
## Accounts / Credentials

| Username   | Password | Status | Notes |
|------------|----------|--------|-------|
| REG-999999 | secret | Active | The user is active and can be used to access the service |
| TEST1-DK   | secret | Active | Not authorized, the user does not have registrator status |
| REG-123456 | secret | Active | The users password is temporary and cannot be used to access service. |

<a name="references"></a>
# References

Here is a list of documents and references used in this document

* General Terms and Conditions: https://www.dk-hostmaster.dk/fileadmin/filer/pdf/generelle_vilkaar/general-conditions.pdf
* RFC: 2616 Hypertext Transfer Protocol -- HTTP/1.1: https://tools.ietf.org/html/rfc2616
* RFC: 2617 HTTP Authentication: Basic and Digest Access Authentication: http://tools.ietf.org/html/rfc2617
* Documentation on the format of a domain name with the DK Hostmaster A/S registry: https://www.dk-hostmaster.dk/english/technical-administration/forms/register-domainname/

<a name="resources"></a>
# Resources

<a name="a-list-of-resources-for-dk-hostmaster-das-service-support-is-listed-below"></a>
Resources for DK Hostmaster DAS support is listed below.

<a name="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster DAS implementation. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

* `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a name="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the DAS implementation or sandbox or production environments, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

 * `info@dk-hostmaster.dk`

<a name="additional-information"></a>
## Additional Information

The DK Hostmaster website:

  * `https://www.dk-hostmaster.dk/en/das`

<a name="demo-client"></a>
## Demo Client

A [demo client](https://github.com/DK-Hostmaster/das-demo-client-mojolicious) is available as open source under a MIT license. 

<a name="appendices"></a>
# Appendices

<a name="http-status-codes"></a>
## HTTP Status Codes

| Status code | Message | Description |
|-------------|---------|-------------|
| 200 | OK | Service returned a valid response |
| 400 | Bad request | The request could not be fulfilled due to missing parameters or malformed syntax |
| 401 | Unauthorized | Authentication failed |
| 403 | Forbidden | Not authorized |
| 404 | Page not found | The request assumes a service (URL) not provided or unsupported at this time |
| 415 | Unsupported Media Type | The requested media type is unsupported, see section on Media Types |
| 429 | Too many attempts | Rate limiting triggered, please see section on Rate Limiting |
| 500 | Server Error | Service malfunction |
| 503 | Service Unavailable | Maintenance mode | 

[general_terms_and_conditions]: https://www.dk-hostmaster.dk/en/general-conditions
