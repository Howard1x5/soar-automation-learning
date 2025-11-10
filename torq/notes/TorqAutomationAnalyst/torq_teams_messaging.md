Microsoft Teams – Messaging & Adaptive Cards

Purpose:
Integrate Microsoft Teams messaging into Torque workflows, enabling rich formatting, adaptive cards, and interactive user prompts.
Key Messaging Capabilities

    Standard Text Messages with Markdown formatting:

        Bold, Italic, bullet/numbered lists

        Nested lists

        Tables

        Hyperlinks

        Code blocks

    Adaptive Cards for interactive layouts and data collection

    Ask a Question functionality for user input directly in Teams

    Conversation Targeting:

        Direct message to a user (email)

        Message a channel/group

        Respond to a specific conversation ID (from workflow event)

Workflow Setup

    Select Template

        Start from the Microsoft Teams Messaging template in Torque.

        Pre-configured with examples for markdown, adaptive cards, and questions.

    Configure Integration

        Set integration name (e.g., "Torque Microsoft Teams").

        Define message target:

            Team + Channel Name

            User Email

            Conversation ID (from event context)

    Trigger Workflow

        Create a filter for event_text or similar field (e.g., "Example 2").

        When triggered, the workflow sends a Teams message using the provided target info.

Example: Sending a Formatted Message

**Title:** Workflow Execution Summary
- Step 1: Completed successfully
- Step 2: Failed (missing input)

[View Logs](https://example.com/logs)

    Use Teams-supported Markdown for clean presentation.

    Keep important info concise and scannable.

Ask a Question in Teams

    Step: Add “Ask a Question” node in workflow.

    Target: Use the same conversation_id from the triggering event.

    Options:

        Buttons (Yes/No)

        Free text input

        People picker

    Bot Auto-Install: Enable "auto_install": true to ensure the bot is installed if not already.

    Timeout: Set duration (e.g., 5 minutes) for user response.

    Default Value: Optional — specify a fallback if no response.

Workflow Response Handling:

if response == "Yes":
    proceed_to_next_step()
else:
    handle_decline()

Adaptive Cards

    Use JSON-based Adaptive Card schema for richer messages.

    Can contain:

        Text blocks

        Images

        Action buttons

        Input fields

    Responses are returned to the workflow as structured JSON.

    Preview & design at: Adaptive Cards Designer

Best Practices

    Thread Responses: Keep related messages in the same conversation thread to reduce channel clutter.

    Format for Readability: Use Markdown for clarity and emphasis.

    Validate Inputs: When using “Ask a Question,” handle both expected and unexpected responses.

    Fallback Paths: Always add timeout/default handling to avoid workflows stalling indefinitely.