# n8n Tutorial for Beginners: Building Your First AI Agent

This tutorial guides you through creating your first AI agent using **n8n**, a no-code automation platform, based on the concepts from the provided video transcript. The agent will handle scheduling, reply to leads, and book meetings automatically using drag-and-drop nodes. We'll fill in gaps from the video, such as specific node configurations, system prompts, and additional practical details for beginners.

---

## Prerequisites
Before starting, ensure you have:
- An **n8n account** (sign up at [n8n.io](https://n8n.io/) or use the cloud version).
- A **Google account** for calendar and spreadsheet integration.
- An **OpenAI API key** (not ChatGPT Plus). Sign up at [platform.openai.com](https://platform.openai.com/), add funds, and generate an API key.
- Basic understanding of Google Calendar and Google Sheets.

---

## Step 1: Set Up Your n8n Workflow
1. **Log in to n8n**:
   - Access n8n via the cloud or your local installation.
   - Click **New Workflow** and name it "Appointment Agent."

2. **Understand the Workspace**:
   - The n8n canvas is your workspace, where nodes (building blocks) are connected to form the agent's logic.
   - Each node performs a specific task, like receiving a message or checking a calendar.

<xaiArtifact>

---

## Step 2: Add a Trigger Node
The trigger starts your workflow when an event occurs, such as receiving a message.

1. **Add a Chat Trigger Node**:
   - In the n8n canvas, click the "+" button to add a node.
   - Search for **Chat Message Received** and drag it to the canvas.
   - This node listens for incoming messages (e.g., "Book a meeting with Laura next Tuesday").

2. **Configuration**:
   - No additional configuration is needed for the basic chat trigger in this tutorial.
   - For testing, n8n provides a chat interface to simulate user input.

---

## Step 3: Add the AI Agent Node (The Brain)
The AI agent node is the decision-making core, powered by a large language model (LLM) like GPT-4.

1. **Add the AI Agent Node**:
   - Click "+" and search for **AI Agent**.
   - Drag it to the canvas and connect it to the Chat Trigger node by dragging the output dot of the trigger to the input dot of the AI Agent.

2. **Configure the AI Agent**:
   - Double-click the AI Agent node to open its settings.
   - **Source for Prompt**: Set to "Connected Chat Trigger Node."
   - **User Message**: Drag the `{{ $json["chatInput"] }}` variable from the right sidebar (this captures the user's message).
   - **System Prompt**: Define the agent's role, tasks, tools, and constraints. Use the following example:

<xaiArtifact artifact_id="cc319e11-9723-4118-8295-0f3ac40a6ae5" artifact_version_id="df6628b8-a471-4dd3-aefe-39760d9dae9f" title="n8n_beginner_tutorial.md" contentType="text/markdown">

```markdown
**System Prompt Example**:
You are an AI scheduling assistant. Your role is to:
- Help users book meetings by checking calendar availability and scheduling events.
- Retrieve contact details from a Google Sheet.
- Respond politely and ask for clarification if needed.

**Tasks**:
1. Interpret user requests for scheduling (e.g., "Book a meeting with Laura next Tuesday at 2 PM").
2. Check Google Calendar for availability.
3. Retrieve contact details from Google Sheets.
4. Schedule the meeting if possible or suggest alternative times.

**Tools**:
- Google Calendar: Use "Get Meetings" to check availability and "Schedule Meeting" to book events.
- Google Sheets: Use to fetch contact details (e.g., email, phone).

**Constraints**:
- Only book meetings if calendar availability is confirmed.
- Verify contact information before scheduling.
- If uncertain (e.g., missing time or contact), ask: "Could you clarify the time or contact name?"
- Never assume details; always confirm with the user.
```

3. **Connect the LLM**:
   - In the AI Agent node, click the "+" to add an LLM.
   - Select **OpenAI** and paste your OpenAI API key.
   - Choose **GPT-4** (or GPT-4o for better performance if available).
   - Save the settings.

---

## Step 4: Set Up Memory
Memory allows the agent to reference past interactions for context.

1. **Add a Simple Memory Node**:
   - Click "+" and search for **Simple Memory**.
   - Drag it to the canvas and connect it to the AI Agent node.

2. **Configure Memory**:
   - Double-click the Simple Memory node.
   - Set **Session ID** to `{{ $json["sessionId"] }}` (drag from the sidebar, linked to the Chat Trigger).
   - Set **Message Context Window** to 5 (this retains the last 5 messages for context).
   - Save the settings.

---

## Step 5: Add Tools (Google Calendar and Google Sheets)
Tools enable the agent to interact with external services like calendars and contact databases.

### Google Sheets (Contacts)
1. **Add a Google Sheets Node**:
   - Click "+" and search for **Google Sheets**.
   - Drag it to the canvas and connect it to the AI Agent node.

2. **Configure Google Sheets**:
   - Double-click the node.
   - **Authenticate**: Connect your Google account and grant access to Google Sheets.
   - **Tool Description**: Set to "Manually" and enter:
     ```
     Call this tool to fetch contact details (e.g., email, phone) from a Google Sheet based on the contact name.
     ```
   - **Resource**: Select "Spreadsheet."
   - **Operation**: Select "Read."
   - **Document**: Choose your Google Sheet (create one with columns: Name, Email, Phone).
   - **Sheet**: Select the specific sheet within the document (e.g., "Sheet1").
   - Save the settings.

### Google Calendar (Get Meetings)
1. **Add a Google Calendar Node**:
   - Click "+" and search for **Google Calendar**.
   - Drag it to the canvas and connect it to the AI Agent node.

2. **Configure Get Meetings**:
   - Double-click the node.
   - **Authenticate**: Connect your Google account and grant access to Google Calendar.
   - **Tool Description**: Set to "Automatically."
   - **Resource**: Select "Event."
   - **Operation**: Select "Get Many."
   - **Calendar**: Choose the specific calendar (e.g., your primary calendar).
   - **After/Before**: Set a time range to check availability (e.g., `{{ $json["startTime"] - 86400000 }}` for one day before, and `{{ $json["startTime"] + 86400000 }}` for one day after, in milliseconds).
   - Save the settings.

### Google Calendar (Schedule Meetings)
1. **Add Another Google Calendar Node**:
   - Click "+" and add another **Google Calendar** node.
   - Connect it to the AI Agent node.

2. **Configure Schedule Meetings**:
   - Double-click the node.
   - **Authenticate**: Use the same Google account.
   - **Tool Description**: Set to "Manually" and enter:
     ```
     Call this tool to schedule a meeting, call, or appointment with a person.
     ```
   - **Resource**: Select "Event."
   - **Operation**: Select "Create."
   - **Calendar**: Choose the same calendar as above.
   - **Start Time**: Set to `{{ $json["startTime"] }}` (user-provided or from prompt).
   - **End Time**: Set to `{{ $json["endTime"] }}` or calculate (e.g., `{{ $json["startTime"] + 3600000 }}` for a 1-hour meeting).
   - **Attendees**: Set to `{{ $json["attendeeEmail"] }}` (fetched from Google Sheets).
   - **Summary**: Set to `{{ $json["eventName"] }}` (e.g., "Meeting with Laura").
   - Save the settings.

---

## Step 6: Add Guardrails
Guardrails prevent errors like hallucinations or incorrect actions.

1. **Update System Prompt**:
   - In the AI Agent node, add the following constraints to the system prompt:
     ```
     - Only reply if calendar availability is confirmed.
     - Never send emails or book meetings without verifying contact information.
     - If uncertain, respond: "I wasn't able to find a matching time or contact. Please try again."
     ```

2. **Optional Conditional Logic**:
   - Add an **If Node** before the Schedule Meeting node.
   - Configure it to check if `{{ $json["attendeeEmail"] }}` exists and if the time slot is free.
   - If false, route to a **Chat Output Node** with a message like: "Please provide a valid contact or choose another time."

---

## Step 7: Test Your Agent
1. **Run the Workflow**:
   - Click **Execute Workflow** in n8n.
   - Open the chat interface (available in n8n's testing panel).

2. **Test Scenarios**:
   - **Normal Case**: "Book a meeting with Laura next Tuesday at 2 PM."
     - The agent should check the calendar, fetch Laura's email from Google Sheets, and book the meeting.
   - **Edge Case 1**: "Book a meeting with Laura" (no time specified).
     - The agent should ask: "Could you clarify the time?"
   - **Edge Case 2**: "Book a meeting at 2 PM next Tuesday" (no contact specified).
     - The agent should ask: "Could you clarify the contact name?"
   - **Edge Case 3**: Book during a busy time slot.
     - The agent should respond: "That time slot is unavailable. Please choose another time."

3. **Debugging**:
   - If the agent fails, check node connections, API keys, and prompt clarity.
   - Ensure Google Sheets and Calendar are correctly authenticated.

---

## Step 8: Deploy and Scale
1. **Deploy the Agent**:
   - Save the workflow and enable it for production use.
   - Share the chat interface with users or integrate it with tools like Slack (add a Slack node for notifications).

2. **Scaling**:
   - Add more tools (e.g., email nodes for confirmation emails).
   - Use HTTP Request nodes to connect to other APIs (e.g., weather or CRM systems).
   - Join the n8n community for templates and advanced workflows.

---

## Tips for Success
- **Start Simple**: Focus on one task (e.g., scheduling) before adding complexity.
- **Test Thoroughly**: Try edge cases to ensure robustness.
- **Learn from the Community**: Join n8n's community or forums for support and inspiration.
- **Iterate**: Remix this agent for other tasks like lead qualification or data summarization.

---

## Example System Prompt (Full)
Below is the complete system prompt for reference:

<xaiArtifact artifact_id="5664a66a-bf54-411f-8019-04935acfa76d" artifact_version_id="95ed7773-621e-464f-9433-bcfda1a8c9ad" title="n8n_beginner_tutorial.md" contentType="text/markdown">

```markdown
You are an AI scheduling assistant. Your role is to:
- Help users book meetings by checking calendar availability and scheduling events.
- Retrieve contact details from a Google Sheet.
- Respond politely and ask for clarification if needed.

**Tasks**:
1. Interpret user requests for scheduling (e.g., "Book a meeting with Laura next Tuesday at 2 PM").
2. Check Google Calendar for availability using the "Get Meetings" tool.
3. Retrieve contact details (e.g., email) from Google Sheets.
4. Schedule the meeting using the "Schedule Meeting" tool if possible or suggest alternative times.

**Tools**:
- Google Calendar: Use "Get Meetings" to check availability and "Schedule Meeting" to book events.
- Google Sheets: Use to fetch contact details (e.g., email, phone).

**Constraints**:
- Only book meetings if calendar availability is confirmed.
- Verify contact information before scheduling.
- If uncertain (e.g., missing time or contact), ask: "Could you clarify the time or contact name?"
- If the time slot is unavailable, respond: "That time slot is unavailable. Please choose another time."
- Never assume details; always confirm with the user.
```

---

## Next Steps
- **Explore n8n**: Check n8n's documentation for advanced nodes and integrations.
- **Join Communities**: Visit n8n's community or the creator's school community for more challenges.
- **Experiment**: Build agents for other tasks like sales follow-ups or data summarization using the same brain-memory-tools framework.

Congratulations! You've built a functional AI agent with n8n. Keep experimenting and leverage this knowledge to create powerful, no-code solutions.