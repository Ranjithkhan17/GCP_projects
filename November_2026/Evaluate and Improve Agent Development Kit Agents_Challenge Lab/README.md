Overview
In this challenge lab, you will act as an engineer on a project to develop an AI agent for Cymbal Pools, a swimming pool installation company. Your goal is to evaluate, debug, and improve an Agent Development Kit (ADK) agent designed to manage pool installation records in Google Cloud using Agents CLI (agents-cli eval).

To achieve this, you will:

Set up a development environment with Agent Development Kit (ADK) and Agents CLI (agents-cli).
Configure custom LLM-as-a-Judge evaluation metrics in tests/eval/eval_config.yaml and build an evaluation dataset in tests/eval/datasets/ to assess specific issues in the agent's behavior.
Run the agents-cli eval pipeline to generate traces and grade the baseline agent.
Improve agent behavior by implementing transaction-safe tools and updating agent instructions.
Re-run evaluations and use agents-cli eval compare to verify the improvements against your baseline.
Challenge Scenario
