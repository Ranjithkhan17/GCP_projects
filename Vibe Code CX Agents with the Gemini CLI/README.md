# 🚀 Vibe Code CX Agents with the Gemini CLI

Vibe coding is an emerging software development practice that uses artificial intelligence (AI) to generate functional code from natural language prompts, accelerating development, and making app building more accessible to developers of all skill levels.

---

## 📋 Objective

In this lab, you learn how to perform the following tasks:

- ✅ Configure and interact with Gemini CLI
- ✅ Configure CX Agent Studio as a Model Context Protocol (MCP) server
- ✅ Vibe code a CX agent using Gemini CLI

---

## # Task 1: Configure Gemini CLI to use the CX Agent Studio MCP server

### Step 1.1: Enable the CX Agent Studio MCP server

In the Google Cloud Console, open Cloud Shell, and run the following command to enable the CX Agent Studio MCP server:

```bash
gcloud beta services mcp enable ces.googleapis.com \
--project $GOOGLE_CLOUD_PROJECT
```

### Step 1.2: List all available tools

List all available tools in the CX Agent Studio MCP server:

```bash
curl -s -X POST "https://ces.googleapis.com/mcp" \
 -H "Content-Type: application/json" \
 -d '{"jsonrpc":"2.0","id":0,"method":"tools/list"}' | jq
```

### Step 1.3: Create a configuration file

Create a config file to configure the CX Agent Studio MCP server:

```bash
mkdir -p ~/.gemini/extensions/cx-agent
cat > ~/.gemini/extensions/cx-agent/gemini-extension.json << EOF
{
  "name": "cx-agent",
  "version": "1.0.0",
  "mcpServers": {
    "cx_agent": {
      "httpUrl": "https://ces.googleapis.com/mcp",
      "authProviderType": "google_credentials",
      "oauth": {
        "scopes": [
          "https://www.googleapis.com/auth/ces"
        ]
      },
      "timeout": 30000,
      "headers": {
        "x-goog-user-project": "$GOOGLE_CLOUD_PROJECT"
      }
    }
  }
}
EOF
```

### Step 1.4: Verify extensions are loaded

In Cloud Shell, run the Gemini CLI and click enter if asked to trust the folder. In Gemini CLI, run the following command to list all extensions available to verify that the CX agents extension is loaded:

```bash
gemini
/extensions list
/mcp list
```

---

## # Task 2: Create a CX agent from the console

1. Open a new tab and navigate to the **CX Agent Studio console**
2. Select your **Project ID** and click **Continue**
3. Ensure that the **Location** is set to **us**
4. Click **Create your first AI agent**
5. Name it **cymbal-taxi** and click **Create**
6. In the right-hand side panel, click on **Global Settings**
7. Click on the **Advanced** tab
8. Scroll down to **Logging** and then click **Enable Cloud Logging**
9. Click **X** to close the side panel

---

## # Task 3: Vibe code your CX agent from the Gemini CLI

In this task, you use the Gemini CLI to vibe code your CX agent by adding tools and instructions using natural language through the MCP server.

### Step 3.1: Return to Gemini CLI

In the first tab, go back to the Gemini CLI. If the session is no longer available, start a new one by running the `gemini` command in the Cloud Shell.

### Step 3.2: List all CX apps

In the Gemini CLI, run the following command to list all CX apps:

```bash
list all cx apps
```

**Prompts you may encounter:**
- If asked to clarify the location, enter: `us`
- If asked permission, select: `Allow for this session`
- If asked to clarify the project, enter: your **Project ID**
- If asked for permission, select: `Allow all server tools for this session`

### Step 3.3: List all CX agents

Now ask Gemini to show you all CX agents for your app:

```bash
list all cx agents for the previously listed app
```

### Step 3.4: Create taxi ETA tool

Ask Gemini to vibe code a tool to calculate the taxi ETA for the agent:

```bash
Create a tool for the previously listed cx agent to help customers order a taxi. It should be called `get_taxi_eta`. It should receive a pickup location and a destination and provide the estimated time for the taxi to arrive.
```

### Step 3.5: Create taxi booking tool

Ask Gemini to vibe code a tool to book a taxi for the agent:

```bash
Create a tool for the previously listed cx agent to help customers order a taxi. It should be called `book_taxi`. It should receive a pickup location and a destination and provide a confirmation message with the booking details.
```

### Step 3.6: Update agent with instructions

Ask Gemini to vibe code the agent for you by providing instructions in natural language:

```bash
update the previously listed cx agent to help customers order a taxi. It should start the conversation by introducing itself as the Cymbal Taxi assistant. It should ask the customer for the pickup location, then ask for the destination. After that, it should use the get_taxi_eta tool to provide the estimated time, and then ask for confirmation before using the book_taxi tool to complete the booking.
```

---

## # Task 4: Test the Cymbal Taxi agent assistant

1. Navigate to the **CX Agent Studio console** and select your **Project ID**
2. Click on **cymbal-taxi** to open your agent
3. Review the newly created agent, the instructions, and the two tools that were created
4. In the right-hand side panel, click on **Global Settings**
5. Click on the **Advanced** tab
6. Scroll down to **Logging** and verify that **Enable Cloud Logging** is still enabled; otherwise, enable it
7. Click **X** to close the side panel

### Preview Agent Conversation

Go to **Preview Agent** and run the following conversation:

| User Input | Agent Response |
|:---|:---|
| Hi | Hello! I'm the Cymbal Taxi assistant. I can help you order a taxi. Where would you like to be picked up? |
| Alexanderplatz, Berlin | Great, thank you. And where will you be going? |
| Brandenburg Gate | I have a taxi for you going from Alexanderplatz, Berlin to Brandenburg Gate. The estimated pickup time is 11 minutes. Should I go ahead and book it for you? |
| Yes | Your taxi from Alexanderplatz, Berlin to Brandenburg Gate has been booked. |

---

## 🎉 Congratulations!

You have successfully completed the lab! You've learned how to:
- ✨ Configure the Gemini CLI with CX Agent Studio MCP server
- ✨ Create a CX agent using the console
- ✨ Vibe code tools and agent instructions using natural language
- ✨ Test your agent using the Preview Agent interface

---

## 📚 Additional Resources

- [Google Cloud CX Agent Studio Documentation](https://cloud.google.com/documentation/cx-agents)
- [Gemini CLI Documentation](https://cloud.google.com/docs/gemini-cli)
- [Model Context Protocol (MCP) Reference](https://modelcontextprotocol.io/)