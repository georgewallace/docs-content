---
navigation_title: "Synthetic monitor status"
---

# Create a synthetic monitor status rule [observability-monitor-status-alert]


Within the Synthetics UI, create a **Monitor Status** rule to receive notifications based on errors and outages.

1. To access this page, go to **Synthetics** → **Overview**.
2. At the top of the page, click **Alerts and rules** → **Monitor status rule** → **Create status rule**.


## Filters [observability-monitor-status-alert-filters]

The **Filter by** section controls the scope of the rule. The rule will only check monitors that match the filters defined in this section. In this example, the rule will only alert on `browser` monitors located in `Asia/Pacific - Japan`.

:::{image} ../../../images/serverless-synthetic-monitor-filters.png
:alt: Filter by section of the Synthetics monitor status rule
:class: screenshot
:::


## Conditions [observability-monitor-status-alert-conditions]

Conditions for each rule will be applied to all monitors that match the filters in the [**Filter by** section](../../../solutions/observability/incident-management/create-monitor-status-rule.md#observability-monitor-status-alert-filters). You can choose the number of times the monitor has to be down relative to either a number of checks run or a time range in which checks were run, and the minimum number of locations the monitor must be down in.

::::{note}
Retests are included in the number of checks.

::::


The **Rule schedule** defines how often to evaluate the condition. Note that checks are queued, and they run as close to the defined value as capacity allows. For example, if a check is scheduled to run every 2 minutes, but the check takes longer than 2 minutes to run, a check will not run until the previous check has finished.

You can also set **Advanced options** such as the number of consecutive runs that must meet the rule conditions before an alert occurs.

In this example, the conditions will be met any time a `browser` monitor is down `3` of the last `5` times the monitor ran across any locations that match the filter. These conditions will be evaluated every minute, and you will only receive an alert when the conditions are met three times consecutively.

:::{image} ../../../images/serverless-synthetic-monitor-conditions.png
:alt: Filters and conditions defining a Synthetics monitor status rule
:class: screenshot
:::


## Action types [observability-monitor-status-alert-action-types]

Extend your rules by connecting them to actions that use the following supported built-in integrations.

* [Cases](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/cases-action-type.md)
* [D3 Security](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/d3security-action-type.md)
* [Email](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/email-action-type.md)
* [{{ibm-r}}](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/resilient-action-type.md)
* [Index](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/index-action-type.md)
* [Jira](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/jira-action-type.md)
* [Microsoft Teams](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/teams-action-type.md)
* [Observability AI Assistant](asciidocalypse://docs/kibana/docs/reference/connectors-kibana/obs-ai-assistant-action-type.md)
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


After you select a connector, you must set the action frequency. You can choose to create a summary of alerts on each check interval or on a custom interval. For example, send email notifications that summarize the new, ongoing, and recovered alerts each hour:

:::{image} ../../../images/serverless-synthetic-monitor-action-types-summary.png
:alt: synthetic monitor action types summary
:class: screenshot
:::

Alternatively, you can set the action frequency such that you choose how often the action runs (for example, at each check interval, only when the alert status changes, or at a custom action interval). In this case, you must also select the specific threshold condition that affects when actions run: the *Synthetics monitor status* changes or when it is *Recovered* (went from down to up).

:::{image} ../../../images/serverless-synthetic-monitor-action-types-each-alert.png
:alt: synthetic monitor action types each alert
:class: screenshot
:::

You can also further refine the conditions under which actions run by specifying that actions only run when they match a KQL query or when an alert occurs within a specific time frame:

* **If alert matches query**: Enter a KQL query that defines field-value pairs or query conditions that must be met for notifications to send. The query only searches alert documents in the indices specified for the rule.
* **If alert is generated during timeframe**: Set timeframe details. Notifications are only sent if alerts are generated within the timeframe you define.

:::{image} ../../../images/serverless-synthetic-monitor-action-types-more-options.png
:alt: synthetic monitor action types more options
:class: screenshot
:::


### Action variables [observability-monitor-status-alert-action-variables]

Use the default notification message or customize it. You can add more context to the message by clicking the icon above the message text box and selecting from a list of available variables.

:::{image} ../../../images/serverless-synthetic-monitor-action-variables.png
:alt: synthetic monitor action variables
:class: screenshot
:::

The following variables are specific to this rule type. You an also specify [variables common to all rules](../../../explore-analyze/alerts-cases/alerts/rule-action-variables.md).

`context.checkedAt`
:   Timestamp of the monitor run.

`context.hostName`
:   Hostname of the location from which the check is performed.

`context.lastErrorMessage`
:   Monitor last error message.

`context.locationId`
:   Location id from which the check is performed.

`context.locationName`
:   Location name from which the check is performed.

`context.locationNames`
:   Location names from which the checks are performed.

`context.message`
:   A generated message summarizing the status of monitors currently down.

`context.monitorId`
:   ID of the monitor.

`context.monitorName`
:   Name of the monitor.

`context.monitorTags`
:   Tags associated with the monitor.

`context.monitorType`
:   Type (for example, HTTP/TCP) of the monitor.

`context.monitorUrl`
:   URL of the monitor.

`context.reason`
:   A concise description of the reason for the alert.

`context.recoveryReason`
:   A concise description of the reason for the recovery.

`context.status`
:   Monitor status (for example, "down").

`context.viewInAppUrl`
:   Open alert details and context in Synthetics app.
