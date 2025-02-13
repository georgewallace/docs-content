---
mapped_urls:
  - https://www.elastic.co/guide/en/kibana/current/action-types.html
  - https://www.elastic.co/guide/en/serverless/current/action-connectors.html
applies:
  serverless: all
  hosted: all
  ece: all
  eck: all
  stack: all
---

# Manage connectors [connector-management]

$$$connector-management$$$

Connectors serve as a central place to store connection information for both Elastic and third-party systems. They enable the linking of actions to rules, which execute as background tasks on the {{kib}} server when rule conditions are met. This allows rules to route actions to various destinations such as log files, ticketing systems, and messaging tools. Different {{kib}} apps may have their own rule types, but they typically share connectors. The **{{stack-manage-app}} > {{connectors-ui}}** provides a central location to view and manage all connectors in the current space.

::::{note}
This page is about Kibana connectors that integrate with services like generative AI model providers. If you’re looking for Search connectors that synchronize third-party data into {{es}}, refer to [Connector clients](https://www.elastic.co/guide/en/serverless/current/elasticsearch-ingest-data-through-integrations-connector-client.html).

::::

:::{image} ../images/kibana-connector-listing.png
:alt: Example connector listing in the {{rules-ui}} UI
:class: screenshot
:::

## Available connectors [action-types]

{{kib}} provides the following types of connectors:

::::{note}
The available connectors varies by project type when using {{serverless}}.
::::

* [{{bedrock}}](https://www.elastic.co/guide/en/kibana/current/bedrock-action-type.html): Send a request to {{bedrock}}.
* [Cases](https://www.elastic.co/guide/en/kibana/current/cases-action-type.html): Add alerts to cases.
* [CrowdStrike](https://www.elastic.co/guide/en/kibana/current/crowdstrike-action-type.html): Send a request to CrowdStrike.
* [D3 Security](https://www.elastic.co/guide/en/kibana/current/d3security-action-type.html): Send a request to D3 Security.
* [{{gemini}}](https://www.elastic.co/guide/en/kibana/current/gemini-action-type.html): Send a request to {{gemini}}.
* [Email](https://www.elastic.co/guide/en/kibana/current/email-action-type.html): Send email from your server.
* [{{ibm-r}}](https://www.elastic.co/guide/en/kibana/current/resilient-action-type.html): Create an incident in {{ibm-r}}.
* [Index](https://www.elastic.co/guide/en/kibana/current/index-action-type.html): Index data into Elasticsearch.
* [Jira](https://www.elastic.co/guide/en/kibana/current/jira-action-type.html): Create an incident in Jira.
* [Microsoft Teams](https://www.elastic.co/guide/en/kibana/current/teams-action-type.html): Send a message to a Microsoft Teams channel.
* [Observability AI Assistant](https://www.elastic.co/guide/en/kibana/current/obs-ai-assistant-action-type.html): Add AI-driven insights and custom actions to your workflow.
* [OpenAI](https://www.elastic.co/guide/en/kibana/current/openai-action-type.html): Send a request to OpenAI.
* [{{opsgenie}}](https://www.elastic.co/guide/en/kibana/current/opsgenie-action-type.html): Create or close an alert in {{opsgenie}}.
* [PagerDuty](https://www.elastic.co/guide/en/kibana/current/pagerduty-action-type.html): Send an event in PagerDuty.
* [SentinelOne](https://www.elastic.co/guide/en/kibana/current/sentinelone-action-type.html): Send a request to SentinelOne.
* [ServerLog](https://www.elastic.co/guide/en/kibana/current/server-log-action-type.html): Add a message to a Kibana log.
* [{{sn-itsm}}](https://www.elastic.co/guide/en/kibana/current/servicenow-action-type.html): Create an incident in {{sn}}.
* [{{sn-sir}}](https://www.elastic.co/guide/en/kibana/current/servicenow-sir-action-type.html): Create a security incident in {{sn}}.
* [{{sn-itom}}](https://www.elastic.co/guide/en/kibana/current/servicenow-itom-action-type.html): Create an event in {{sn}}.
* [Slack](https://www.elastic.co/guide/en/kibana/current/slack-action-type.html): Send a message to a Slack channel or user.
* [{{swimlane}}](https://www.elastic.co/guide/en/kibana/current/swimlane-action-type.html): Create an incident in {{swimlane}}.
* [{{hive}}](https://www.elastic.co/guide/en/kibana/current/thehive-action-type.html): Create cases and alerts in {{hive}}.
* [Tines](https://www.elastic.co/guide/en/kibana/current/tines-action-type.html): Send events to a Tines Story.
* [Torq](https://www.elastic.co/guide/en/kibana/current/torq-action-type.html): Trigger a Torq workflow.
* [{{webhook}}](https://www.elastic.co/guide/en/kibana/current/webhook-action-type.html): Send a request to a web service.
* [{{webhook-cm}}](https://www.elastic.co/guide/en/kibana/current/cases-webhook-action-type.html): Send a request to a Case Management web service.
* [xMatters](https://www.elastic.co/guide/en/kibana/current/xmatters-action-type.html): Send actionable alerts to on-call xMatters resources.

::::{note}
Some connector types are paid commercial features, while others are free. For a comparison of the Elastic subscription levels, go to [the subscription page](https://www.elastic.co/subscriptions).

::::

## Required permissions [_required_permissions_2]

Access to connectors is granted based on your privileges to alerting-enabled features. For more information, go to [Security](../explore-analyze/alerts-cases/alerts/alerting-setup.md#alerting-security).

## Connector networking configuration [_connector_networking_configuration]

Use the [action configuration settings](https://www.elastic.co/guide/en/kibana/current/alert-action-settings-kb.html#action-settings) to customize connector networking configurations, such as proxies, certificates, or TLS settings. You can set configurations that apply to all your connectors or use `xpack.actions.customHostSettings` to set per-host configurations.

## Connector list [connectors-list]

In **{{stack-manage-app}} > {{connectors-ui}}**, you can find a list of the connectors in the current space. You can use the search bar to find specific connectors by name and type. The **Type** dropdown also enables you to filter to a subset of connector types.

:::{image} ../../../images/kibana-connector-filter-by-type.png
:alt: Filtering the connector list by types of connectors
:class: screenshot
:::

You can delete individual connectors using the trash icon. Alternatively, select multiple connectors and delete them in bulk using the **Delete** button.

:::{image} ../../../images/kibana-connector-delete.png
:alt: Deleting connectors individually or in bulk
:class: screenshot
:::

::::{note}
You can delete a connector even if there are still actions referencing it. When this happens the action will fail to run and errors appear in the {{kib}} logs.

::::

## Creating a new connector [creating-new-connector]

New connectors can be created with the **Create connector** button, which guides you to select the type of connector and configure its properties.

:::{image} ../../../images/kibana-connector-select-type.png
:alt: Connector select type
:class: screenshot
:::

After you create a connector, it is available for use any time you set up an action in the current space.

For out-of-the-box and standardized connectors, refer to [preconfigured connectors](https://www.elastic.co/guide/en/kibana/current/pre-configured-connectors.html).

::::{tip}
You can also manage connectors as resources with the [Elasticstack provider](https://registry.terraform.io/providers/elastic/elasticstack/latest) for Terraform. For more details, refer to the [elasticstack_kibana_action_connector](https://registry.terraform.io/providers/elastic/elasticstack/latest/docs/resources/kibana_action_connector) resource.
::::

## Importing and exporting connectors [importing-and-exporting-connectors]

To import and export connectors, use the [Saved Objects Management UI](/explore-analyze/find-and-organize/saved-objects.md).

:::{image} ../../../images/kibana-connectors-import-banner.png
:alt: Connectors import banner
:class: screenshot
:::

If a connector is missing sensitive information after the import, a **Fix** button appears in **{{connectors-ui}}**.

:::{image} ../../../images/kibana-connectors-with-missing-secrets.png
:alt: Connectors with missing secrets
:class: screenshot
:::

## Monitoring connectors [monitoring-connectors]

The [Task Manager health API](../../../deploy-manage/monitor/kibana-task-manager-health-monitoring.md) helps you understand the performance of all tasks in your environment. However, if connectors fail to run, they will report as successful to Task Manager. The failure stats will not accurately depict the performance of connectors.

For more information on connector successes and failures, refer to the [Event log index](../../../explore-analyze/alerts-cases/alerts/event-log-index.md).
