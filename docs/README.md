# Reporting API Events Integration

## Overview

The Reporting API Events integration allows you to monitor security and observability events emitted by browsers.

Browsers have mixed support for a number of reporting APIs:

* report-uri
* report-to
* reporting-endpoints

Each of these provide a hint for browsers to optionally implement, which describes a URL as a reporting endpoint for notable events:

* Content-Security-Policy Violations
* Network Error Logs
* Crash Logs
* Browser Interventions
* Deprecation Warnings
* Permissions Policy Violations

Collecting and monitoring these events provides valuable insight into browser and device behavior _outside of the normal browser scope_, e.g. to detect DNS errors or malicious JavaScript injection.

This integration supports all report types from all API specifications.

## Limitations

Currently the Agent integration cannot exist independently, as the filebeat `http_endpoint` module does not support the HTTP OPTIONS preflight request mechanism.
Until [#43930](https://github.com/elastic/beats/issues/43930) is resolved you will require a reverse proxy or edge function / worker to respond correctly to browser preflight requests. The issue above documents the requirements.

If you choose to add the `logs-reporting_api.*` data stream to the Observability application settings for error documents, clicking on an error group from the Observability application will currently result in an error. This is because the Reporting API events cannot be associated with any specific transaction - the APIs are privacy-preserving and all events are anonymous.

Browser support is complex. Some browsers will include a `user-agent` header in their reports, others won't. Some will send permissions policy violation reports, others won't. The reports received by this integration are to be considered a sample and not representative of all browsers.

**Importantly** this is not an official Elastic integration and no warrantee or support is offered at this time.

## Requirements

* At least one publicly-accessible host on which to install Elastic Agent and the Reporting API Events integration
* DNS name with valid TLS certificate, or a CDN property, which resolve to the host(s) with the integration installed
* Ability to update response headers on HTML documents

## Setup

Firstly, download the latest release version from [GitHub](https://github.com/simonhearne/reporting_api) as a zip archive. Then upload to Kibana using the following cURL command or equivalent with a Kibana user with [required privileges](https://www.elastic.co/docs/reference/fleet/fleet-roles-privileges):

```cURL
curl -XPOST \             
  -H 'content-type: application/zip' \
  -H 'kbn-xsrf: true' \
  https://<kibana-host>/api/fleet/epm/packages \
  -u <username>:<password> \
  --data-binary @reporting_api-<version>.zip
```

1. Add the integration to an agent policy with at least one agent. Add one integration per service name, multiple integrations can run on the same agent host.
1. Create a DNS name which resolves to the agent host(s), optionally via a content delivery network to simplify TLS configuration
1. Create or update the relevant response headers on your HTML documents (e.g. `report-to`, `report-uri`, `reporting-endpoints`) to include the new reporting endpoint, including the relevant path if configured in the integration
1. Validate that events are appearing in the `logs-reporting_api.*` data stream
1. (optionally) update the `Indices` [settings](https://www.elastic.co/docs/solutions/observability/apm/applications-ui-settings#apm-indices-settings) in the Observability application to include the `logs-reporting_api.*` data stream in the `Error Indices` input

**Temporarily**: a CDN configuration or reverse proxy is required to return a valid response to preflight requests from browsers, as documented in [#43930](https://github.com/elastic/beats/issues/43930).

There are a number of configuration items available in the integration:

* **Enable Observability Error Fields**: when enabled, the integration will create all fields required for Reporting API Events to appear in the Observability application for the defined service name.
* **Enable HTTP Tracer**: this setting enables local logs of HTTP requests on the Agent filesystem for debugging.
* **Remove Agent Fields**: this setting removes metadata fields which are not required for the integration, including details about the Agent hosting environment.

## References

Specifications:

* [report-uri](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/report-uri)
* [report-to](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/report-to)
* [Reporting API](https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API)
* [Reporting Endpoints](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Reporting-Endpoints)
* [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy#directives)
* [Network Error Logging](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Network_Error_Logging)
* [Permissions Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Permissions-Policy)

Example HTML document response headers using all API versions for maximum compatibility:

```http
content-security-policy: default-src 'self' data: 'unsafe-inline'; report-uri https://<reporting-endpoint>[/<path>]; report-to default;
nel:                     {"report_to": "default","max_age": 31536000, "include_subdomains": true, "success_fraction": 1.0, "failure_fraction": 1.0}
permissions-policy:      accelerometer=(),geolocation=*
report-to:               {"group":"default","max_age":31536000,"endpoints":[{"url":"https://<reporting-endpoint>[/<path>]"}],"include_subdomains":true}
reporting-endpoints:     default="https://<reporting-endpoint>[/<path>]"
```

## Roadmap

* [ ]  Add comprehensive tests
* [ ]  Add data view and saved searches
* [ ]  Add Dashboards to integration package (Overview, CSP, NEL, Crash, Permissions Policy, Deprecations, etc.)
* [ ]  Add Stacktraces where appropriate

## Logs

### reporting_api

Data stream of Reporting API Events.

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| cloud.image.id | Image ID for the cloud instance. | keyword |
| container.labels | Image labels. | object |
| data_stream.dataset | Data stream dataset name. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| ecs.version | ECS version used in the event | keyword |
| error.culprit | Culprit responsible for the error | keyword |
| error.exception.message | Exception message for the error | text |
| error.grouping_key | Grouping key for the error | keyword |
| error.grouping_name | Grouping name for the error | text |
| error.id | Unique identifier for the error | keyword |
| error.type | Type of error (e.g., csp-report, crash) | keyword |
| event.created | Timestamp when the event was created | date |
| event.dataset | Event dataset | constant_keyword |
| event.ingested | Timestamp when the event was ingested | date |
| event.module | Event module | constant_keyword |
| event.original | Original event message before processing | text |
| host.containerized | If the host is a container. | boolean |
| host.os.build | OS build information. | keyword |
| host.os.codename | OS codename, if any. | keyword |
| input.type | Type of Filebeat input. | keyword |
| message | Human-readable message describing the event | text |
| observer.type | Type of observer (e.g., reporting-api) | keyword |
| processor.event | Processor event type (e.g., error) | keyword |
| reporting_api.body.blockedURL | Blocked URL from the CSP report | keyword |
| reporting_api.body.disposition | Disposition from the CSP report | keyword |
| reporting_api.body.documentURL | Document URL from the CSP report | keyword |
| reporting_api.body.effectiveDirective | Effective directive from the CSP report | keyword |
| reporting_api.body.originalPolicy | Original policy from the CSP report | text |
| reporting_api.body.referrer | Referrer URL from the CSP report | keyword |
| reporting_api.body.sample | Script sample from the CSP report | text |
| reporting_api.body.sourceFile | Source file from the CSP report | keyword |
| reporting_api.body.statusCode | Status code from the CSP report | integer |
| reporting_api.type | Type of Reporting API event (e.g., csp-report, crash) | keyword |
| reporting_api.url | URL associated with the Reporting API event | keyword |
| service.environment | APM service environment this integration is linked to | keyword |
| service.name | APM service name this integration is linked to | keyword |
| tags | Tags associated with the event | keyword |
| url.full | Full URL associated with the event | keyword |
| url.original | Original URL associated with the event | keyword |
| user_agent.original | Original user agent string | keyword |

