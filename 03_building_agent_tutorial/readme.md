# n8n AI Agent Building Tutorial

[Watch Tutorial](https://youtu.be/geR9PeCuHK4?si=SGGNqGNw4Ge_L81n)

This tutorial is designed to take you from a beginner with no prior knowledge of AI agents to a proficient builder using n8n, a powerful no-code automation platform. Based on a comprehensive YouTube course, this guide provides step-by-step instructions to create automations and AI agents, focusing on practical applications without requiring coding expertise. By the end, you'll have the tools, knowledge, and confidence to build fully autonomous AI agents.

---

## Section 1: Understanding AI Agents and Automations

### What Are AI Agents?
AI agents are systems that act autonomously, using tools to perform complex tasks requiring human-like reasoning. Unlike automations, which follow predefined workflows for consistent tasks, AI agents adapt dynamically to new information and unique scenarios.

- **Key Difference**: Automations are static and repeatable (e.g., watering a garden at the same time daily), while AI agents adjust based on dynamic inputs (e.g., adjusting water flow based on weather, soil moisture, or plant type).
- **Why AI Agents Are Powerful**: They replace human intervention in tasks, adapt to edge cases, and operate tirelessly, making them ideal for efficiency and scalability.

### Who Are AI Agents For?
AI agents benefit everyone, from marketers and construction workers to data analysts and CEOs. They save time, increase efficiency, and create new income streams by automating repetitive tasks or building sellable systems. Examples include:
- Saving time on daily tasks.
- Creating new business opportunities by selling AI agent workflows.
- Enhancing efficiency in industries like marketing, construction, and e-commerce.

### Tools Needed
This tutorial uses two primary tools:
1. **n8n**: A no-code platform for building automations and agents, integrating with over 500 tools (e.g., Google, Microsoft, Slack). Start with a 14-day free trial at [n8n.io](https://n8n.io).
2. **OpenAI**: Provides the AI model (e.g., GPT-4o mini) for intelligent decision-making. Requires an API key with $5-$10 credit.

---

## Section 2: Getting Started with n8n

### Setting Up n8n
1. **Create an Account**:
   - Visit [n8n.io](https://n8n.io) and click "Get Started" in the top right.
   - Enter your name, email, and password, then select "Start 14-day Free Trial."
   - Optionally, self-host n8n for free using the community edition (requires technical setup).
2. **Pricing Overview**:
   - **Starter Plan**: $24/month (or annually with a discount).
   - **Pro Plan**: $60/month, includes advanced features.
   - **Enterprise Plan**: Custom pricing for teams.
   - A 14-day free trial is available for all plans.
3. **Navigating the n8n Interface**:
   - **Workspace**: A clean canvas for building workflows.
   - **Sidebar**: Click the "+" button to create workflows, credentials, or projects.
   - **Projects and Folders**: Organize workflows into projects (e.g., "Full Agents Guide") and folders (e.g., "Section 1: Basics").
   - **Templates**: Browse pre-made workflows for inspiration (free and paid).
   - **Settings**: Manage account details, enable 2FA, or add team members (enterprise features).

### Understanding Workflows
- Workflows are the backbone of automations and agents, consisting of nodes that perform actions.
- Create a workflow by clicking "+" > "New Workflow," then save and name it (e.g., "Workflow 1").
- Organize workflows in folders or projects for clarity.

---

## Section 3: Mastering n8n Nodes

Nodes are the building blocks of n8n workflows, each performing a specific action. There are five node types:

1. **Trigger Nodes**:
   - Start workflows based on events (e.g., schedule, new email, Telegram message).
   - Example: A "Schedule" node runs daily at 9 AM, or a "Gmail" node triggers when a new email arrives.
2. **Action Nodes**:
   - Perform tasks in external apps (e.g., send an email, create a Google Sheets row).
   - Example: A "Gmail" node can send a message or reply to a thread.
3. **Utility Nodes**:
   - Transform data with filters, if-statements, or conversions.
   - Example: Filter data to route specific information to different apps.
4. **Code Nodes**:
   - Run custom JavaScript or HTTP requests for advanced customization.
   - Example: Make an API call to fetch external data.
5. **Advanced AI Nodes**:
   - Integrate AI for autonomous decision-making (e.g., sentiment analysis, custom prompts).
   - Example: Use an "AI Agent" node to process queries with OpenAI’s GPT-4o mini.

### Exploring Nodes
- **Trigger Examples**:
  - **Manual Trigger**: Start a workflow manually.
  - **Schedule**: Run daily at a specific time.
  - **Webhook**: Trigger when an external event occurs.
  - **Chat**: Start when a message is sent to n8n’s native chat.
- **Action Examples**:
  - **Gmail**: Send, reply, or mark emails as read/unread.
  - **Google Sheets**: Create or update rows.
  - **Airtable**: Manage records in a database.
- **Utility Examples**:
  - Filter data by specific conditions.
  - Convert data formats for compatibility.
- **Code Examples**:
  - Run JavaScript to manipulate data.
  - Use HTTP requests to connect to unsupported APIs.
- **AI Examples**:
  - Sentiment analysis to detect positive/negative text.
  - AI Agent node to process natural language queries.

---

## Section 4: Building Your First Automation

### Automation: Lead Form Notification
This automation takes form submissions from a project’s lead form and sends an email notification.

1. **Create a New Workflow**:
   - In your n8n workspace, go to your project folder (e.g., "Full Agents Guide" > "Section 1: Basics").
   - Click "+" > "New Workflow" and name it "Lead Form Automation."
2. **Add a Trigger Node**:
   - Click "Add First Step" > select "Webhook" node (simulates a form submission).
   - Configure the webhook to accept POST requests with form data (e.g., name, email).
   - Rename the node to "Form Submission" for clarity.
3. **Add an Action Node**:
   - Click "+" > select "Gmail" node > "Send a Message."
   - Connect your Gmail account (create a test account if needed).
   - Configure the node to send an email to yourself with the subject "New Lead!" and body containing form data (e.g., "New lead: {{ $node['Form Submission'].json['name'] }}").
   - Rename the node to "Send Email."
4. **Test the Automation**:
   - Save the workflow and click "Execute Workflow."
   - Send a test POST request to the webhook URL (use a tool like Postman or a form builder).
   - Verify that you receive an email with the lead details.
5. **Tips for Success**:
   - Start with tasks you understand well (e.g., processes you’ve done manually).
   - Test incrementally to ensure each node works as expected.
   - Organize workflows clearly to avoid clutter.

---

## Section 5: Building a Multi-Agent Hierarchy

### Multi-Agent Personal Assistant
This section builds a multi-agent system that handles Google Calendar and email tasks using a classifier agent to route queries to specialized agents.

1. **Create a Project Structure**:
   - In n8n, create a folder named "Personal Assistant" in your project.
   - Inside, create a subfolder named "Tools" for specialized agents.
   - Create a workflow named "Classifier Agent" in the "Personal Assistant" folder.
2. **Set Up the Classifier Agent**:
   - **Trigger Node**: Add a "Chat" node as the communication medium (rename to "Chat").
   - **AI Agent Node**: Add an "AI Agent" node, connect it to the Chat node, and select "GPT-4o mini" as the chat model (rename to "4.1 Mini").
   - **System Message**:
     ```plaintext
     You are a personal assistant, an expert at determining which tool is best needed for the job and then passing user queries to those specific tools. The two tools you have access to are Google Calendar Agent and Emailing Agent. The user will be giving commands or asking questions about either their email or Google Calendar. Your job is to pass two variables to each of these tools: the user query and a description of the task needing completed. You are to create the description to clearly define what the user needs so the other agents can interpret the user better. For example, if the user creates an event and wants to email someone about it, then you should coordinate the communication between the two agents: the Emailing Agent and the Google Calendar Agent.
     ```
   - Save the workflow.
3. **Create the Google Calendar Agent**:
   - In the "Tools" folder, create a workflow named "Google Calendar Agent."
   - **Trigger Node**: Add an "Execute Subworkflow" node, configured to trigger when called by another workflow. Define two input fields:
     - `userQuery` (string)
     - `jobDescription` (string)
     - Rename to "Personal Assistant Data."
   - **AI Agent Node**: Add an "AI Agent" node, connect it to the trigger, and select "GPT-4o mini." Add a system message:
     ```plaintext
     Today's date is {{ $now }}. You are an expert Google Calendar manager. Your job is to take in userQuery and jobDescription to determine which tool needs to be used in order to complete what is being requested. If you don't have sufficient information to complete the task, ask the user for more details in the category you need more details in.
     ```
   - **Prompt**: Map the inputs dynamically:
     ```plaintext
     userQuery: {{ $node["Personal Assistant Data"].json["userQuery"] }}
     jobDescription: {{ $node["Personal Assistant Data"].json["jobDescription"] }}
     ```
   - **Tools**:
     - Add a "Google Calendar" tool for "Create Event":
       - Name: `create_events`
       - Description: "Create an event in Google Calendar."
       - Connect your Google Calendar account.
       - Let the model define parameters like start time, end time, and summary (title).
     - Add a "Google Calendar" tool for "Get Many Events":
       - Name: `search_all_events`
       - Description: "Get many events in Google Calendar."
       - Enable "Return All" and let the model define parameters.
     - Add a "Google Calendar" tool for "Update Event":
       - Name: `update_events`
       - Description: "Update an event in Google Calendar."
       - Add fields for summary, start time, and end time, letting the model define them.
     - Add a "Google Calendar" tool for "Delete Event":
       - Name: `delete_events`
       - Description: "Delete an event in Google Calendar."
       - Let the model define the event ID.
     - Add a "Google Calendar" tool for "Get Event":
       - Name: `get_single_event`
       - Description: "Retrieve a single event in Google Calendar."
       - Let the model define the event ID.
   - Save the workflow.
4. **Connect the Classifier Agent to the Google Calendar Agent**:
   - In the "Classifier Agent" workflow, add a tool: "Call n8n Workflow."
   - Name: `Google Calendar Agent`
   - Description: "Call this tool to do anything regarding Google Calendar."
   - Select the "Google Calendar Agent" workflow.
   - Let the model define `userQuery` and `jobDescription`.
   - Save the workflow.
5. **Test the System**:
   - In the "Classifier Agent" workflow, open the Chat node and enter: "Create an event for tomorrow at 2 PM that lasts for an hour titled movie date."
   - Execute the workflow. The classifier should pass the query to the Google Calendar Agent, which creates the event.
   - Check your Google Calendar to confirm the event.
   - Test additional queries:
     - "What do I have on Wednesday on my schedule?"
     - "Change the movie date to be an hour later."
     - "Create an event for tomorrow." (The agent should ask for more details.)
   - Verify the results in your calendar and the chat responses.

---

## Section 6: Your Task – Build the Emailing Agent

To solidify your learning, extend the multi-agent system by creating an Emailing Agent that integrates with the Classifier Agent.

1. **Create the Emailing Agent Workflow**:
   - In the "Tools" folder, create a workflow named "Emailing Agent."
   - **Trigger Node**: Add an "Execute Subworkflow" node, configured to accept `userQuery` and `jobDescription` as inputs. Rename to "Personal Assistant Data."
   - **AI Agent Node**: Add an "AI Agent" node with "GPT-4o mini." Add a system message:
     ```plaintext
     You are an expert email manager. Your job is to take in userQuery and jobDescription to determine which tool needs to be used to complete what is being requested. If you don't have sufficient information to complete the task, ask the user for more details in the category you need more details in.
     ```
   - **Prompt**: Map the inputs:
     ```plaintext
     userQuery: {{ $node["Personal Assistant Data"].json["userQuery"] }}
     jobDescription: {{ $node["Personal Assistant Data"].json["jobDescription"] }}
     ```
   - **Tools**:
     - Add a "Gmail" tool for "Send a Message":
       - Name: `send_email`
       - Description: "Send an email via Gmail."
       - Connect your Gmail account.
       - Let the model define parameters like recipient, subject, and body.
     - Add a "Gmail" tool for "Get a Message":
       - Name: `get_email`
       - Description: "Retrieve an email from Gmail."
       - Let the model define the message ID.
2. **Connect to the Classifier Agent**:
   - In the "Classifier Agent" workflow, add a "Call n8n Workflow" tool.
   - Name: `Emailing Agent`
   - Description: "Call this tool to do anything regarding email."
   - Select the "Emailing Agent" workflow.
   - Let the model define `userQuery` and `jobDescription`.
3. **Test the Emailing Agent**:
   - In the "Classifier Agent" workflow, use the Chat node to enter: "Send an email to test@example.com with the subject 'Meeting Tomorrow' and body 'Hi, let's meet at 2 PM.'"
   - Execute and verify the email is sent.
   - Test additional queries like: "Get my latest email."
   - Refine the system by iterating on instructions or tools if needed.

---

## Tips for Success
- **Start Small**: Begin with simple automations to understand node connections.
- **Iterate**: Test and refine workflows to handle edge cases.
- **Organize**: Use folders and projects to keep workflows manageable.
- **Leverage Community**: Join the AI Foundations community for support, templates, and advanced courses (link in the original video description).
- **Practice**: Build and test workflows in a test environment before deploying.

---

## Next Steps
- Experiment with additional tools (e.g., Slack, Airtable) to expand your agents’ capabilities.
- Explore n8n’s template library for pre-built workflows.
- Consider advanced features like code nodes or sentiment analysis for more complex agents.
- Share your progress or issues in the comments of the original video for community support.

This tutorial provides the foundation to build powerful AI agents with n8n. By mastering nodes, triggers, and multi-agent hierarchies, you can automate tasks, save time, and even create sellable systems. Happy building!