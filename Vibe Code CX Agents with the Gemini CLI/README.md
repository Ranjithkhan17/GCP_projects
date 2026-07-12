Vibe coding is an emerging software development practice that uses artificial intelligence (AI) to generate functional code from natural language prompts, accelerating development, and making app building more accessible, especially for those with limited programming experience.
-------------------------------------------------------------------------------------------------------------------
Objective
In this lab, you learn how to perform the following tasks:

Configure and interact with Gemini CLI.
Configure CX Agent Studio as a Model Context Protocol (MCP) server.
Vibe code a CX agent using Gemini CLI.


Task 1. Configure Gemini CLI to use the CX Agent Studio MCP server
In the Google Cloud Console, open Cloud Shell, and run the following command to enable the CX Agent Studio MCP server.

```
gcloud beta services mcp enable ces.googleapis.com \
--project $GOOGLE_CLOUD_PROJECT
```

List all available tools in the CX Agent Studio MCP server.
```
curl -s -X POST "https://ces.googleapis.com/mcp" \
 -H "Content-Type: application/json" \
 -d '{"jsonrpc":"2.0","id":0,"method":"tools/list"}' | jq
```
Create a config file to configure the CX Agent Studio MCP server.

```
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
In Cloud Shell, run the Gemini CLI and click enter if asked to trust the folder.In Gemini CLI, run the following command to list all extensions available to verify that the CX agents extension is loaded.List all available tools in the CX Agent Studio MCP server /mcp list
```
gemini
/extensions list
/mcp list
```

Task 2. Create a CX agent from the console
Open a new tab and navigate to the CX Agent Studio console, select your Project ID, and then click Continue.

Ensure that the Location is set to us.

Click Create your first AI agent.

Name it cymbal-taxi and click Create.

In the right-hand side panel, click on Global Settings.

Click on the Advanced tab.

Scroll down to Logging and then click Enable Cloud Logging.

Click X to close the side panel.

Task 3. Vibe code your CX agent from the Gemini CLI

In this task, you use the Gemini CLI to vibe code your CX agent by adding tools and instructions using natural language through the MCP server.

In the first tab, go back to the Gemini CLI. If the session is no longer available, start a new one by running the gemini command in the Cloud Shell.

In the Gemini CLI, run the following command to list all CX apps.

```
list all cx apps
```

If asked to clarify the location, enter us.
If asked permission, select Allow for this session.
If asked to clarify the project, enter your Project ID.
If asked for permission, select Allow all server tools for this session.

Now ask gemini to show you all CX agents for your app.

```
list all cx agents for the previously listed app
```

Ask gemini to vibe code a tool to calculate the taxi ETA for the agent
```
Create a tool for the previously listed cx agent to help customers order a taxi. It should be called `get_taxi_eta`. It should receive a pickup location and a destination and provide the estimated time of pickup. For calculating the estimated time of pickup, give a random number between 5 and 15 minutes.
```
Ask gemini to vibe code a tool to book a taxi for the agent.
```
Create a tool for the previously listed cx agent to help customers order a taxi. It should be called `book_taxi`. It should receive a pickup location and a destination and provide a confirmation message that the taxi has been booked and a boolean saying booked: true.
```
Ask gemini to vibe code the agent for you by providing instructions in natural language.
```
update the previously listed cx agent to help customers order a taxi. It should start the conversation by introducing itself as the Cymbal Taxi assistant. It should ask the customer for the pickup location and the destination. Once it has all the information, it should provide the estimated time of pickup, which will be calculated using the `get_taxi_eta` tool, and ask to confirm the order. Once confirmed, it should call the `book_taxi` tool to book the taxi.
```

Task 4. Test the Cymbal Taxi agent assistant

Navigate to the CX Agent Studio console and select your Project ID, and click on cymbal-taxi to open your agent.

Review the newly created agent, the instruction and the two tools that were created.

In the right-hand side panel, click on Global Settings.

Click on the Advanced tab.

Scroll down to Logging and verify that Enable Cloud Logging is still enabled; otherwise, enable it.

Click X to close the side panel.

Go to Preview Agent and run the following conversation.

| User Input       | Agent Response |
| --------------- | --------------- |
| Hi              | Hello! I'm the Cymbal Taxi assistant. I can help you order a taxi. Where would you like to be picked up?  |

|Alexanderplatz, 
Berlin            |Great, thank you. And where will you be going?  |


| Brandenburg Gate   | I have a taxi for you going from Alexanderplatz, Berlin to Brandenburg Gate. The estimated pickup time is 11 minutes. Should I go ahead and book it for                         you?   |

| yes   | Your taxi from Alexanderplatz, Berlin to Brandenburg Gate has been booked.   |












