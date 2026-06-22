# Update the requirements.txt with below ( adk version 2.0)
bs4>=0.0.2
beautifulsoup4>=4.13.4
chainlit>=2.11.1
google-cloud-aiplatform[agent_engines,adk]>=1.112
google-adk==2.0

----------------------------------------
#Task 3. Debug the Paint Agent

In the root_agent's file adk_challenge_lab/paint_agent/agent.py, add an AgentTool() to the root_agent's list of tools. Provide the AgentTool() the following arguments:

agent should be set to search_agent (the sub-agent that uses the search tool you uncovered above).
skip_summarization should be set to False because you would like the agent to report on what the search tool has returned.
Remove that sub-agent from the sub_agents list.

Line no:66 yo 70, remove and add below

sub_agents=[room_planner_agent],
    tools=[
        set_session_value,
        AgentTool(agent=search_agent, skip_summarization=False)  
         ],


-------------------------------------
#To deploy ADk 

adk deploy agent_engine paint_agent \
--display_name "Paint Agent" \
--staging_bucket gs://qwiklabs-gcp-02-xxxxxxxxxxxxxxxx( update your bucket)

