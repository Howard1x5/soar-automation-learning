Slack – Message Blocks (Advanced Messaging with Block Kit)

Purpose:
Leverage Slack's Block Kit to send advanced, interactive messages from Torque workflows. Block messages allow you to build structured layouts, interactive forms, and rich message components beyond plain Markdown or text.
Key Concepts

    Block Message: JSON-based structure defining text, buttons, inputs, and other interactive components.

    Block Kit Builder: Official Slack UI tool for designing and previewing block layouts.
    Search: “Slack Block Kit Builder” → Use builder to create and export JSON payloads.

    Block ID: Unique identifier for each input or element inside a block.

        Prevents ambiguity when multiple inputs exist.

        Makes parsing workflow responses predictable.

Basic Workflow Steps

    Prepare JSON Payload

        Use Block Kit Builder to design your message.

        Copy JSON output (strip sensitive IDs if sharing).

        Example minimal form payload:

    {
      "blocks": [
        {
          "type": "input",
          "block_id": "token_verification",
          "element": {
            "type": "plain_text_input",
            "action_id": "token_input"
          },
          "label": {
            "type": "plain_text",
            "text": "Enter Token Verification Code"
          }
        }
      ]
    }

Send Block Message

    In Torque, add a Send Message Block step.

    Paste JSON payload into the block message content.

    Select target Slack channel or user.

Capture Responses

    When user submits, Slack sends block form data back to workflow.

    Response JSON structure:

input_values → block_id → action_id → value

If Block ID is set to "token_verification", the returned value will be:

        input_values.token_verification.token_input.value

    Why Use Block IDs

        Avoids needing to reference by block index (block[0], block[1]), which can break if order changes.

        Ensures consistent mapping between user input and workflow variables.

Example Use Case: Token Verification Prompt

    Goal: Request a token from a user and store it for verification.

    Steps:

        Send a block form with one text input labeled "Enter Token Verification Code".

        Wait for user submission.

        Retrieve the value from input_values.token_verification.token_input.value.

        Pass it to verification logic or API call in workflow.

Tips for Building Block Messages

    Use Block Kit Builder for rapid prototyping.

    Combine multiple block types (buttons, select menus, sections) for richer interactivity.

    Keep messages threaded when part of an ongoing conversation to reduce channel clutter.

    Test each payload in a dedicated Slack test channel before production use.