Slack Messaging – Torque Academy Notes

Overview:
This module covers sending messages from Torque workflows directly to Slack (and optionally Microsoft Teams), including formatting with Markdown, sending threaded messages, sending snippets for large data, and asking interactive questions with buttons.
Key Concepts

    Triggering Workflows via Keywords

        Example keyword: LL1

        When Slack message text contains the keyword, the workflow executes.

        Useful for testing and controlling when workflows fire.

    Slack Markdown Formatting

        Supports:

            Italics → _italic text_

            Bold → *bold text*

            Block quotes → > quote

            Links → <https://url|display_text>

            Code blocks → ```code```

        Enhances message clarity and presentation.

    Using Slack Event Data

        Events contain channel_id, user_id, and thread_ts (timestamp).

        Use Get User Info step to enrich with the real username (real_name).

    Sending Messages

        Send directly to:

            A user (user_id)

            A channel (channel_id)

        Use Thread TS to keep replies grouped.

    Threaded Messaging

        Add thread_ts parameter to keep conversations tidy.

        Helps avoid clutter in active channels.

    Sending Large Data (Snippets)

        Slack message limit: ~3,000 characters.

        For CSV/JSON or long text, use send snippet step.

    Interactive Questions

        Use Ask a Question block to present buttons or dropdowns.

        Capture user responses for conditional logic.

        Options include:

            Default values

            Timeout settings

            Allow “None” responses

Slack → Torque Example Flow

    Trigger on keyword (Slack2).

    Get channel and user info from event.

    Send formatted Markdown message.

    Thread reply using thread_ts.

    Optionally send a snippet for long content.

    Ask a question (buttons/dropdowns).

    Use response in workflow logic.

Cross-Platform Messaging Cheat Sheet
Feature	Torque (Slack)	Shuffle (Slack)	Splunk SOAR	XSOAR
Trigger by keyword	Slack Event → text contains keyword	Slack Event → Filter step	Slack App Event → Filter playbook	Slack Integration → Filter trigger
Get User Info	Get User Info step	Slack → Get User Info	Slack App → users.info API	slack-get-user-info automation
Send Markdown	Send Message w/ Markdown	Same	Slack → chat.postMessage	Slack → send-message command
Thread replies	Add thread_ts	Add thread_ts	chat.postMessage thread_ts param	message_id / thread_ts
Send large data	Send Snippet	Send File → Snippet	files.upload API	Slack → upload-file
Ask a question	Ask a Question step	Interactive Message	Block Kit prompt	Slack ask user command
Practice Lab – Shuffle Implementation

Goal: Rebuild the Torque Slack Messaging example in Shuffle for hands-on testing without needing Torque access.

    Setup:

        Sign up for Shuffle (free cloud or local install).

        Create a Slack app & token for testing.

    Workflow Steps:

        Trigger: Slack event → keyword match (e.g., LL1).

        Action: Slack → Send Message (Markdown format).

        Action: Slack → Send Snippet (if > 3,000 chars).

        Action: Slack → Ask Question (buttons).

        Logic: Use “If” conditions based on user answer.

    Test:

        Post keyword in Slack channel.

        Confirm workflow triggers.

        Check formatting, snippet handling, and question logic.