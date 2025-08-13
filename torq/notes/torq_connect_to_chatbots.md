ChatBots – Slack Integration with Torq
Overview

This module covers integrating Slack with Torq for two-way chatbot communication.
Torq also supports Microsoft Teams, Zoom, WebEx, and other popular chat platforms.
Setting up the Slack Integration

    In Torq Integrations:

        Select Slack → click Add.

        Authorize Torq to connect with Slack (OAuth flow).

        If needed, Reauthorize from the integration settings later.

    Customize Bot Appearance:

        Change bot name, logo, and assigned channels.

    Documentation:

        Torq provides setup docs with exact Slack API scopes required.

Sending a Message to Slack

    Workflow Setup:

        Trigger: Manual or scheduled.

        Step: “Send Message” to Slack.

    Parameters:

        Recipient: Slack user’s email address or Slack user_id.

        Message Body: Plain text (or JSON for formatting).

    Execution:

        Message appears in Slack DM.

        Replying from Slack can be captured if workflow includes inbound event handling.

Receiving Messages from Slack

    Trigger:

        “On-demand trigger from Slack” — auto-created when Slack integration is set up.

    Example Condition:

        If inbound message text contains hello, reply with:

        I received your message.

    Publishing:

        Must publish workflow for it to run automatically.

Channel Messaging

    You can send to Slack channels by using the channel name with # prefix.

    Add Torq bot to the channel first.

    Example:

        Send message to #alerts when workflow detects a phishing email.

Running Workflows from Slack

    Use the command:

    torq run <workflow_name>

    Must be signed in via Slack → Torq bot connection.

    Displays list of available workflows you can execute from Slack directly.

ChatOps Cross-Platform Cheatsheet
Feature / Concept	Torq (Slack)	Microsoft Teams	Zoom Chat / WebEx Teams	Shuffle / XSOAR / Splunk SOAR
Bot Setup	OAuth + Torq integration	App registration in Azure AD + API perms	JWT / OAuth app in dev portal	API connector or custom webhook
Trigger on Message	Yes – event triggers in workflows	Yes – Bot Framework webhook	Limited – requires chat app API	Webhook-based
Reply in DM	Yes – direct send to user	Yes – proactive message	Yes – via chat send API	Yes
Reply in Channel	Yes – send to #channel	Yes – channel message	Yes – group chat ID	Yes
Command Execution	torq run <workflow> from chat	Slash commands or messaging extensions	Slash commands / app commands	Custom slash command simulation
Auth Requirements	Slack OAuth + Scopes	Azure AD + Bot Framework token	JWT or OAuth	API key or OAuth
Practice Lab – ChatOps without Torq

Goal: Recreate two-way bot communication in another environment (e.g., Shuffle + Slack API).

    Create Slack App:

        Go to Slack API – Create App

        Enable Bot Token Scopes: chat:write, chat:write.public, channels:history, im:history.

        Install app to workspace.

    Outbound Message:

        In Shuffle, use Slack Send Message action → target user or channel.

    Inbound Event Handling:

        Configure Slack Event Subscription:

            Event: message.im (direct message)

            URL: Shuffle webhook.

    Trigger Workflow:

        If inbound message contains “hello” → respond with “I received your message.”

    Channel Demo:

        Post alerts to #test-channel when triggered manually.