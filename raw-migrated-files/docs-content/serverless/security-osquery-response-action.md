# Add Osquery Response Actions [security-osquery-response-action]

::::{warning}
This functionality is in technical preview and may be changed or removed in a future release. Elastic will work to fix any issues, but features in technical preview are not subject to the support SLA of official GA features.
::::


Osquery Response Actions allow you to add live queries to custom query rules so you can automatically collect data on systems the rule is monitoring. Use this data to support your alert triage and investigation efforts.

::::{admonition} Requirements
:class: note

* Osquery Response Actions require the Endpoint Protection Complete [project feature](../../../deploy-manage/deploy/elastic-cloud/project-settings.md).
* The [Osquery manager integration](../../../solutions/security/investigate/manage-integration.md) must be installed.
* {{agent}}'s [status](asciidocalypse://docs/docs-content/docs/reference/ingestion-tools/fleet/monitor-elastic-agent.md) must be `Healthy`. Refer to [{{fleet}} Troubleshooting](../../../troubleshoot/ingest/fleet/common-problems.md) if it isn’t.
* You must have the appropriate user role to use this feature.
* You can only add Osquery Response Actions to custom query rules.

::::


:::{image} ../../../images/serverless--osquery-available-response-actions-osquery.png
:alt: The Osquery response action
:class: screenshot
:::


## Add Osquery Response Actions to rules [add-osquery-response-action]

You can add Osquery Response Actions to new or existing custom query rules. Queries run every time the rule executes.

1. Choose one of the following:

    * **New rule**: When you are on the last step of [custom query rule](../../../solutions/security/detect-and-alert/create-detection-rule.md#create-custom-rule) creation, go to the Response Actions section and click the **Osquery** icon.
    * **Existing rule**: Edit the rule’s settings, then go to the **Actions** tab. In the tab, click the **Osquery** icon under the Response Actions section.

        ::::{note}
        If the rule’s investigation guide is using an Osquery query, you’ll be asked if you want to add the query as an Osquery Response Action. Click **Add** to add the investigation guide’s query to the rule’s Osquery Response Action.

        ::::

2. Specify whether you want to set up a single live query or a pack:

    * **Query**: Select a saved query or enter a new one. After you enter the query, you can expand the **Advanced** section to set a timeout period for the query, and view or set [mapped ECS fields](../../../solutions/security/investigate/osquery.md#osquery-map-fields) included in the results from the live query. Mapping ECS fields is optional.

        ::::{note}
        Overwriting the query’s default timeout period allows you to support queries that take longer to run. The default and minimum supported value for the **Timeout** field is `60`. The maximum supported value is `900`.

        ::::


        ::::{tip}
        You can use [placeholder fields](../../../solutions/security/investigate/use-placeholder-fields-in-osquery-queries.md) to dynamically add alert data to your query.

        ::::

    * **Pack**: Select from available query packs. After you select a pack, all of the queries in the pack are displayed.

        ::::{tip}
        Refer to [prebuilt packs](../../../solutions/security/investigate/osquery.md#osquery-prebuilt-packs-queries) to learn about using and managing Elastic prebuilt packs.

        ::::


        :::{image} ../../../images/serverless--osquery-setup-single-query.png
        :alt: Shows how to set up a single query
        :class: screenshot
        :::

3. Click the **Osquery** icon to add more live queries (optional).
4. Click **Create & enable rule** (for a new rule) or **Save changes** (for existing rules) to finish adding the queries.


## Edit Osquery Response Actions [edit-osquery-response-action]

If you want to choose a different query or query pack for the Osquery Response Action to use, edit the rule to update the Response Action.

::::{important}
If you edited a saved query or query pack that an Osquery Response Action is using, you must reselect the saved query or query pack on the related Osquery Response Action. Query changes are not automatically applied to Osquery Response Actions.

::::


1. Edit the rule’s settings, then go to the **Actions** tab.
2. Modify the settings for Osquery Response Actions you’ve added.
3. Click **Save changes**.


## Find query results [find-osquery-response-action-results]

When a rule generates an alert, Osquery automatically collects data on the host. Query results are displayed within the **Response Results** tab in the Alert details flyout. The number next to the **Response Results** tab represents the number of queries attached to the rule, in addition to endpoint response actions run by the rule.

::::{note}
Refer to [Examine Osquery results](../../../solutions/security/investigate/examine-osquery-results.md) for more information about query results.

::::


:::{image} ../../../images/serverless--osquery-osquery-results-tab.png
:alt: Shows how to set up a single query
:class: screenshot
:::
