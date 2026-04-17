{{- generatedHeader }}
{{/*
This template can be used as a starting point for writing documentation for your new integration. For each section, fill in the details
described in the comments.

Find more detailed documentation guidelines in https://www.elastic.co/docs/extend/integrations/documentation-guidelines
*/}}
# Devolutions Server Integration for Elastic

## Overview
The Devolutions Server integration for Elastic collects Devolutions Server audit and operational syslog events and maps them to the Elastic Common Schema (ECS).
This integration facilitates authentication monitoring, privileged access auditing, shared connection activity tracking, and operational troubleshooting for Devolutions Server deployments.

### Compatibility
This integration is compatible with Devolutions Server environments that emit syslog records in the RFC5424-style format represented by the sample events bundled with this package.
The package supports Elastic Stack and Kibana versions that satisfy the package manifest requirement.

### How it works
This integration receives Devolutions Server logs over syslog using Elastic Agent `TCP` or `UDP` inputs.
The ingest pipeline parses the common log header, routes each event to a family-specific sub-pipeline, and maps authentication, session management, personal connection, and administrative activity to ECS.

## What data does this integration collect?
The Devolutions Server integration collects log messages of the following types:
* Login and authentication events.
* Session management events such as profile updates.
* Personal connection audit events such as connection opens, entry views, credential access, and entry modifications.
* Administrative and system events such as cleanup jobs, application start, migrations, and syslog heartbeat messages.

### Supported use cases
* Audit Devolutions Server logins and identify the user, client application, client version, and source IP involved in each login.
* Track privileged access activity for shared entries, passwords, domains, and connection launches.
* Detect changes to Devolutions entries, user-specific settings, and password material.
* Troubleshoot server-side operational events such as migrations, cleanup jobs, startup events, and heartbeat records.

## What do I need to use this integration?
To use this integration, you need the following:
* Administrative access to the Devolutions Server deployment so you can configure syslog forwarding.
* Network connectivity from Devolutions Server to the Elastic Agent host on the selected syslog port.
* An enrolled Elastic Agent with this integration added to an agent policy.
* A decision on whether to use `TCP` for more reliable delivery or `UDP` for lower-overhead transport.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

Elastic Agent is required to stream data from the syslog receiver and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Onboard / configure
1. In Kibana, open **Management > Integrations** and add the **Devolutions Server** integration to the target Elastic Agent policy.
2. Choose the syslog transport you want to expose from the agent policy:
	* `TCP` when delivery guarantees and back-pressure matter more than simplicity.
	* `UDP` when lower overhead is preferred and occasional packet loss is acceptable.
3. Configure the listener host and port in the integration policy. The package defaults to port `9514`.
4. In Devolutions Server Web-UI, [configure syslog forwarding](https://docs.devolutions.net/server/web-interface/administration/configuration/server-settings/general/logging/#syslog-server) inside `Administration > Server settings > Logging` so the service sends logs to the Elastic Agent host and port you selected. Ensure the protocol matches the input you configured in Kibana.
5. If you are validating parsing or troubleshooting format mismatches, enable **Preserve original event** so the raw syslog payload is copied to `event.original`.
6. Save the policy and confirm the Elastic Agent becomes healthy in Fleet.

### Validation
1. Generate one or more Devolutions Server events, such as a successful login, a personal connection open, or an administrative maintenance action.
2. In Kibana, open **Discover** and filter on `data_stream.dataset: "devolutions_server.log"`.
3. Confirm events are arriving and verify representative fields such as `event.action`, `event.category`, `user.name`, `source.ip`, `devolutions_server.service`, and `observer.ip`.
4. If parsing does not look correct, inspect `event.original` and `error.message` to see which branch or processor failed.

## Troubleshooting

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

Common issues for this integration include the following:
* Port or protocol mismatch between Devolutions Server and the Elastic Agent input.
* Local firewalls or network ACLs blocking the configured syslog port.
* Older or customized syslog formats that do not match the parser included in this package.
* Missing `event.original` during parser debugging because **Preserve original event** was left disabled.
* Timestamp interpretation problems caused by unexpected time zones in the upstream syslog emitter.

## Scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.
Use `TCP` when you need more reliable delivery semantics and `UDP` when you want simpler transport with lower overhead.
For higher-volume environments, distribute syslog senders across multiple Elastic Agents or place a load balancer in front of a horizontally scaled ingest tier.

## Reference
{{/* Repeat for each data stream of the current type
### {Data stream name}

The `{data stream name}` data stream provides events from {source} of the following types: {list types}.

For each data_stream_name, include an optional summary of the datastream, the exported fields reference table and the sample event.

The fields template function will be replaced by a generated list of all fields from the `fields/` directory of the data stream when building the integration.

#### {data stream name} fields

To include a generated list of fields from the `fields/` directory, uncomment and use:
{{ fields "data_stream_name" }}

The event template function will be replace by a sample event, taken from `sample_event.json`, when building this integration.

To include a sample event from `sample_event.json`, uncomment and use:
{{ event "data_stream_name" }}

*/}}

### log

The `log` data stream provides Devolutions Server authentication, session management, personal connection, and administrative/system events.

#### log fields

{{ fields "log" }}

{{/*
#### log sample event

{{ event "log" }}
*/}}

### Inputs used
{{/* All inputs used by this package will be automatically listed here. */}}
{{ inputDocs }}
