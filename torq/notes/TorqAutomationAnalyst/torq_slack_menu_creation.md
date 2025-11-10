Slack – User-Driven Menu System (Ask a Question + Dynamic Options)

Purpose:
Create a native, user-driven menu system in Torque using Slack's Ask a Question step. This allows dynamically generated options based on user input, executing different workflow branches depending on the selected option, and removing completed options from the list until the menu is empty.
Workflow Overview

    Initialize Options

        Use Set Variable step to define your options list (e.g., ["Option 1", "Option 2", "Option 3"]).

        Store in variable options.

        Keep a variable name for Ask a Question responses (e.g., result) so you can filter later.

    Ask a Question

        Ask a Question → Button type.

        Instead of hard-coding options, use the variable options from the previous step.

        Add a None option (allow_none = true) for users to exit early.

        Set timeout (e.g., 5 minutes) and default response as needed.

    Loop Until Menu is Empty

        Use a Loop with a safe range (e.g., 1–5 for up to 4 choices + None).

        Inside loop:

            Ask a Question using current options list.

            Switch on the response:

                None → Exit loop and send closing message.

                Option 1 / Option 2 / Option 3 → Run corresponding logic (command, nested workflow, API call, etc.).

            Filter Array to remove the selected option from the options list.

    Filter Array Logic

        Field Path: response from Ask a Question step.

        Removes the chosen option from the array.

        Updates options variable so the next loop iteration only shows remaining options.

    Exit Condition

        After filtering, check array length.

        If length == 0, send final "Workflow complete" message and exit.

Example Flow Structure

[Set Variable: options = ["Option 1", "Option 2", "Option 3"]]
    ↓
[Loop: 1–5]
    ↓
[Ask a Question (Slack)]
    ↓
[Switch on Response]
    ├─ None → Send "Exiting menu" → Exit Loop
    ├─ Option 1 → Run Option 1 logic
    ├─ Option 2 → Run Option 2 logic
    └─ Option 3 → Run Option 3 logic
    ↓
[Filter Array: Remove chosen option]
    ↓
[If array length == 0] → Send "All options completed" → Exit Loop

Key Benefits

    Dynamic Menus: Options update automatically as user progresses.

    No Hard-Coding: One workflow can adapt to different scenarios by setting options list at runtime.

    Slack Native: Works directly in Slack channels or threads for a clean user experience.

    Reusable: Can be adapted for Microsoft Teams or other chat integrations with Ask a Question equivalents.

