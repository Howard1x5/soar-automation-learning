Microsoft Teams – Adaptive Cards & Forms

Purpose:
Use Microsoft Teams Adaptive Cards to create rich, interactive forms and messages within Torque workflows. Adaptive Cards enable structured, user-friendly data collection and visually appealing messages.
Two Primary Adaptive Card Actions in Torque

    Post Adaptive Card

        Sends a formatted Adaptive Card message to a Teams channel, group, or user.

        Static display: No form inputs required.

    Send Adaptive Card Form

        Interactive form for collecting multiple inputs in a single card.

        Can include text fields, date pickers, toggles, and dropdowns.

When to Use Each
Action	Best For
Post Adaptive Card	Announcements, status updates, quick reference cards.
Send Adaptive Card Form	Collecting multiple related pieces of data from a user in one interaction.
Basic Adaptive Card Workflow Setup

    Trigger Workflow
    Define your event source and conversation target:

        Team + Channel

        User Email

        Conversation ID (from event context)

    Choose Card Type

        For a static card, use Post Adaptive Card.

        For a form, use Send Adaptive Card Form.

    Design the Card

        Open Adaptive Cards Designer.

        Choose “Microsoft Teams” as the host app for accurate previews.

        Add elements:

            Text Block (for headings/instructions)

            Input Fields (text, date, toggle, choice set)

            Action.Submit (required for forms)

    Assign Field IDs

        Give each input an ID to retrieve its value in the workflow.

        Example: input_string, start_date, priority_toggle

    Copy Card JSON

        Paste the JSON from the designer into the “Card JSON” field of the Torque step.

Example – Simple Text Input Form

Card Layout (JSON snippet):

{
  "type": "AdaptiveCard",
  "body": [
    {
      "type": "TextBlock",
      "text": "Please enter a random string:",
      "size": "Medium",
      "color": "Accent"
    },
    {
      "type": "Input.Text",
      "id": "input_string",
      "placeholder": "Enter value here"
    }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "Submit Your String"
    }
  ],
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.3"
}

Returned Workflow Data Example:

{
  "input_string": "ABC123"
}

Advanced Use Case – User Menu Card

    Present a list of users to select from.

    Retrieve:

        Display name

        Email

        Unique Teams user ID

        Current status

    Useful for admin workflows (assigning tickets, verifying user details, escalating issues).

Best Practices

    Always Test in Teams Preview before deploying to production.

    Use Clear IDs for form elements — avoids confusion when parsing workflow output.

    Validate User Input in subsequent workflow steps.

    Modular Design: Keep reusable card JSON snippets in a shared GitHub folder for quick access.

    Limit Inputs per Card to keep the form readable and reduce user drop-off.

