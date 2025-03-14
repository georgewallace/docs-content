---
navigation_title: "SLO burn rate"
---

# Create a service-level objective (SLO) burn rate rule [slo-burn-rate-alert]


::::{important}
To create and manage SLOs, you need an [appropriate license](https://www.elastic.co/subscriptions), an {{es}} cluster with both `transform` and `ingest` [node roles](asciidocalypse://docs/elasticsearch/docs/reference/elasticsearch/configuration-reference/node-settings.md#node-roles) present, and [SLO access](../../../solutions/observability/incident-management/configure-service-level-objective-slo-access.md) must be configured.

::::


You can create a SLO burn rate rule to get alerts when the burn rate is above a defined threshold for two different lookback periods: a long period and a short period that is 1/12th of the long period. For example, if your long lookback period is one hour, your short lookback period is five minutes.

For each lookback period, the burn rate is computed as the error rate divided by the error budget. When the burn rates for both periods surpass the threshold, an alert is triggered.

::::{note}
When you use the UI to create an SLO, a default SLO burn rate alert rule is created automatically. The burn rate rule will use the default configuration and no connector. You must configure a connector if you want to receive alerts for SLO breaches.
::::


To create an SLO burn rate rule, go to **Observability → SLOs**. Click the more options icon to the right of the SLO you want to add a burn rate rule for, and select **Create new alert rule** from the drop-down menu:

:::{image} ../../../images/observability-create-new-alert-rule-menu.png
:alt: create new alert rule menu
:class: screenshot
:::

To create your SLO burn rate rule:

1. Set your long lookback period under **Lookback period (hours)**. Your short lookback period is set automatically.
2. Set your **Burn rate threshold**. Under this field, you’ll see how long you have until your error budget is exhausted.
3. Set how often the condition is evaluated in the **Check every** field.
4. Optionally, change the number of consecutive runs that must meet the rule conditions before an alert occurs in the **Advanced options**.


## Action types [action-types-slo]

Extend your rules by connecting them to actions that use the following supported built-in integrations. Actions are {{kib}} services or integrations with third-party systems that run as background tasks on the {{kib}} server when rule conditions are met.

You can configure action types on the [Settings](../../../solutions/observability/apps/configure-settings.md#configure-uptime-alert-connectors) page.

* [D3 Security](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/d3security-action-type.md)
* [Email](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/email-action-type.md)
* [{{ibm-r}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/resilient-action-type.md)
* [Index](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/index-action-type.md)
* [Jira](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/jira-action-type.md)
* [Microsoft Teams](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/teams-action-type.md)
* [Observability AI Assistant connector](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/obs-ai-assistant-action-type.md)
* [{{opsgenie}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/opsgenie-action-type.md)
* [PagerDuty](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/pagerduty-action-type.md)
* [Server log](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/server-log-action-type.md)
* [{{sn-itom}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/servicenow-itom-action-type.md)
* [{{sn-itsm}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/servicenow-action-type.md)
* [{{sn-sir}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/servicenow-sir-action-type.md)
* [Slack](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/slack-action-type.md)
* [{{swimlane}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/swimlane-action-type.md)
* [Torq](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/torq-action-type.md)
* [{{webhook}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/webhook-action-type.md)
* [xMatters](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/xmatters-action-type.md)

::::{note}
Some connector types are paid commercial features, while others are free. For a comparison of the Elastic subscription levels, go to [the subscription page](https://www.elastic.co/subscriptions).

::::


After you select a connector, you must set the action frequency. You can choose to create a **Summary of alerts** on each check interval or on a custom interval. For example, you can send email notifications that summarize the new, ongoing, and recovered alerts every twelve hours.

Alternatively, you can set the action frequency to **For each alert** and specify the conditions each alert must meet for the action to run. For example, you can send an email only when the alert status changes to critical.

:::{image} ../../../images/observability-slo-action-frequency.png
:alt: Configure when a rule is triggered
:class: screenshot
:::


## Action variables [action-variables-slo]

Use the default notification message or customize it. You can add more context to the message by clicking the icon above the message text box and selecting from a list of available variables.

:::{image} ../../../images/observability-slo-action-variables.png
:alt: Action variables with default SLO message
:class: screenshot
:::

The following variables are specific to this rule type. You an also specify [variables common to all rules](../../../explore-analyze/alerts-cases/alerts/rule-action-variables.md).

`context.alertDetailsUrl`
:   Link to the alert troubleshooting view for further context and details. This will be an empty string if the `server.publicBaseUrl` is not configured.

`context.burnRateThreshold`
:   The burn rate threshold value.

`context.longWindow`
:   The window duration with the associated burn rate value.

`context.reason`
:   A concise description of the reason for the alert.

`context.shortWindow`
:   The window duration with the associated burn rate value.

`context.sloId`
:   The SLO unique identifier.

`context.sloInstanceId`
:   The SLO instance id.

`context.sloName`
:   The SLO name.

`context.timestamp`
:   A timestamp of when the alert was detected.

`context.viewInAppUrl`
:   The URL to the SLO details page to help with further investigation.


## Alert recovery [recovery-variables-slo]

To receive a notification when the alert recovers, select **Run when Recovered**. Use the default notification message or customize it. You can add more context to the message by clicking the icon above the message text box and selecting from a list of available variables.

:::{image} ../../../images/observability-duration-anomaly-alert-recovery.png
:alt: Default recovery message for Uptime duration anomaly rules with open "Add variable" popup listing available action variables
:class: screenshot
:::


## Next steps [slo-creation-next-steps]

Learn how to view alerts and triage SLO burn rate breaches:

* [View alerts](../../../solutions/observability/incident-management/view-alerts.md)
* [Triage SLO burn rate breaches](../../../solutions/observability/incident-management/triage-slo-burn-rate-breaches.md)

