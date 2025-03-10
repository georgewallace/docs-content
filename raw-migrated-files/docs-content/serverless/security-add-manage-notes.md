# Notes [security-add-manage-notes]

Incorporate notes into your investigative workflows to coordinate responses, conduct threat hunting, and share investigative findings. You can attach notes to alerts, events, and Timelines and manage them from the **Notes** page.

::::{note}
Configure the `securitySolution:maxUnassociatedNotes` [advanced settings](../../../solutions/security/get-started/configure-advanced-settings.md#max-notes-alerts-events) to specify the maximum number of notes that you can attach to alerts and events.

::::



## View and add notes to alerts and events [notes-alerts-events]

Open the alert or event details flyout to access the **Notes** tab, where you can view existing notes and add new ones. To quickly open the tab, click the **Add note** action (![The action that lets you to add a new note](../../../images/serverless-editorComment.svg "")) in the Alerts or Events table. Then, enter a note into the text box, and click **Add note** to create it.

After notes are created, the **Add note** icon displays a notification dot. In the details flyout for alerts, the alert summary in the right panel also shows how many notes are attached to the alert.

:::{image} ../../../images/serverless--notes-new-note-alert-event.png
:alt: New note added to an alert
:::


## View and add notes to Timelines [notes-timelines]

::::{important}
You can only add notes to saved Timelines.

::::


Open the **Notes** Timeline tab, where you can view existing notes for the Timeline and add new ones. Alternatively, use the details flyout for alerts and events that you’re investigating from Timeline. Be aware that notes added this way are automatically attached to the alert or event and the Timeline unless you deselect the **Attach to current Timeline** option.

After notes are created, the **Notes** Timeline tab displays the total number of notes attached to the Timeline.

:::{image} ../../../images/serverless--notes-new-note-timeline-tab.png
:alt: New note added to a Timeline
:::


## Manage notes [manage-notes]

Use the **Notes** page to view and interact with all existing notes. To access the page, navigate to **Investigations*** in the main navigation menu or by using the global search field, then go to ***Notes**. From the **Notes** page, you can:

* Search for specific notes
* Filter notes by the user who created them or by the object they’re attached to (notes can be attached to alerts, events, or Timelines)
* Examine the contents of a note (select the text in the **Note content** column)
* Delete one or more notes
* Examine the alert or event that a note is attached to (click the **Expand alert/event details** ![Preview alert or event details action](../../../images/serverless-expand.svg "") icon)
* Open the Timeline that the note is attached to (click the **Open saved timeline** ![Preview alert or event details action](../../../images/serverless-timelineWithArrow.svg "") icon)

:::{image} ../../../images/serverless--notes-management-page.png
:alt: Notes management page
:::
