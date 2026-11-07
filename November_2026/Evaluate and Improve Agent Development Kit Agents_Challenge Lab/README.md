 Title: Evaluate and Improve Agent Development Kit Agents: Challenge Lab
Overview
In this challenge lab, you will act as an engineer on a project to develop an AI agent for Cymbal Pools, a swimming pool installation company. Your goal is to evaluate, debug, and improve an Agent Development Kit (ADK) agent designed to manage pool installation records in Google Cloud using Agents CLI (agents-cli eval).
To achieve this, you will:
•	Set up a development environment with Agent Development Kit (ADK) and Agents CLI (agents-cli).
•	Configure custom LLM-as-a-Judge evaluation metrics in tests/eval/eval_config.yaml and build an evaluation dataset in tests/eval/datasets/ to assess specific issues in the agent's behavior.
•	Run the agents-cli eval pipeline to generate traces and grade the baseline agent.
•	Improve agent behavior by implementing transaction-safe tools and updating agent instructions.
•	Re-run evaluations and use agents-cli eval compare to verify the improvements against your baseline.
Challenge Scenario
Cymbal Pools is a small business that installs and maintains pools across the United States.
Your company has been brought on to help upgrade their internal systems to make use of new agentic technology.
Your Challenge
You have been handed a prototype ADK agent that manages Cymbal Pools' customer database in BigQuery. As a customer's pool project progresses from an initial quote to final sign-off, the agent is responsible for moving their record between BigQuery tables that represent each stage of the workflow—such as pool_estimates, accepted_with_deposit, denied_estimates, scheduled_installations, completed_pools, and paid_and_closed.
However, initial tests show that the agent has serious logical flaws:
•	Data Inconsistency: It sometimes deletes customer records from one table without adding them to another, causing records to be lost.
•	Invalid Transitions: It moves customers between stages in an invalid order (e.g., marking a pool installation as "paid and closed" before construction is even completed!).
Your challenge is to:
1.	Configure custom evaluation metrics in tests/eval/eval_config.yaml and create an evaluation dataset in tests/eval/datasets/ using agents-cli eval to expose these flaws.
2.	Write robust custom Python tools to enforce consistent transactions and valid transitions.
3.	Restrict the agent's tool access to only use these verified tools.
4.	Re-run agents-cli eval and compare your candidate results against the baseline to prove your fixes work.
Task 1. Install Agents CLI and set up your environment
In this task, you'll set up the Cloud Shell Terminal and Cloud Shell Editor to function as an IDE in Google Cloud, install Agents CLI (agents-cli), download the starter code for the lab, and configure your Python project environment.
In this lab, you will use the uv package manager developed by Astral. uv has become an industry standard to replace pip, venv, and other package and environment management tools. If you are unfamiliar with uv, read Astral's cheat-sheet for projects using uv or work through their series of guides to study it in greater depth.

Install Agents CLI (agents-cli)
1.	In the Cloud Shell Terminal, add ~/.local/bin to your PATH and run uvx google-agents-cli setup to install the agents-cli tool and its bundled agent skills:
2.	export PATH="$HOME/.local/bin:$PATH"
uvx google-agents-cli setup
Download the lab code and configure the project environment
1.	Copy the adk_eval_challenge_lab project directory containing the starter code for this lab from the Cloud Storage bucket and navigate into it:
2.	gcloud storage cp -r gs://YOUR_GCP_PROJECT_ID-bucket/adk_eval_challenge_lab ~
cd ~/adk_eval_challenge_lab
Copied!
3.	Configure your active Google Cloud project, create the .env configuration files for Vertex AI and agents-cli eval, and initialize and activate the uv virtual environment with the required packages from requirements.txt (including google-cloud-aiplatform[evaluation] and google-adk[eval]):
================================================
gcloud config set project YOUR_GCP_PROJECT_ID
cat << EOF > bigquery_agent/.env
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=YOUR_GCP_PROJECT_ID
GOOGLE_CLOUD_LOCATION=global
MODEL=
EOF
cp bigquery_agent/.env .env

uv init
uv add -r requirements.txt
source .venv/bin/activate
======================================================

Task 2. Build and run the evaluation pipeline with Agents CLI Eval
With agents-cli eval, evaluation metrics are configured in tests/eval/eval_config.yaml and evaluation datasets are stored in tests/eval/datasets/. In this task, you will inspect the prototype agent, define custom LLM-as-a-Judge evaluation metrics, create a dataset of test cases, and run a baseline evaluation.
1.	In the Cloud Shell Editor, inspect the following files in the adk_eval_challenge_lab folder:
•	agents-cli-manifest.yaml: Maps agent_directory: "bigquery_agent" so agents-cli eval knows where to find the agent.
•	bigquery_agent/agent.py: Notice that root_agent is currently configured with tools=[bigquery_toolset] and a general SQL prompt—allowing the agent to generate and execute arbitrary INSERT and DELETE queries with no guardrails against data loss or invalid table transitions.
•	tests/eval/eval_config.yaml: Open this file to review and configure the evaluation metrics.
2.	Use the menus to toggle on Word Wrap with View > Word Wrap to make it easier to read and edit the YAML file.
3.	Notice that tests/eval/eval_config.yaml contains two top-level sections:
•	metrics_to_run: The list of built-in or custom metrics that agents-cli eval executes when grading traces.
•	custom_metrics: Custom metric definitions. The file already defines one custom LLM-as-a-Judge metric named ledger_validity, which uses a prompt_template with {prompt}, {response}, and {agent_data} placeholders to evaluate the agent's tool-call trace and reasoning. Notice how ledger_validity directly tests for Data Inconsistency (checking that every deleted record is added to another table in the same interaction).
4.	Add a second custom LLM-as-a-Judge metric named valid_transitions to tests/eval/eval_config.yaml to test for Invalid Transitions (checking whether the agent only moves records between tables in a valid order):
•	Add - valid_transitions under metrics_to_run.
•	Append the valid_transitions metric definition under custom_metrics:
------------------------------------------------------------------------------
- name: valid_transitions
  prompt_template: |
    You are an expert quality assurance evaluator for an AI agent that moves customer records between BigQuery tables.
    Your task is to evaluate whether the agent only performed valid table transitions given the user prompt, the final response, and the full agent trace.

    Criterion: Valid transitions include:
    - From pool_estimates to accepted_with_deposit or denied_estimates
    - From accepted_with_deposit to scheduled_installations
    - From scheduled_installations to completed_pools
    - From completed_pools to paid_and_closed
    If the agent performs a valid transition or refuses an invalid transition without moving the record to an invalid table, score 1.0. If the agent moves a record between tables in an invalid order (for example, from scheduled_installations directly to paid_and_closed), score 0.0.

    ### Target Evaluation:
    User Prompt: {prompt}
    Final Response: {response}
    Full Agent Trace: {agent_data}

    Return your evaluation as a valid JSON object matching this structure:
    {
      "score": 1.0 if all transitions are valid or an invalid transition is refused, 0.0 if an invalid transition is performed,
      "explanation": string explaining the rationale for the score assigned
    }
  judge_model_sampling_count: 1
1.	  judge_model: projects/PROJECT_NUMBER/locations/global/publishers/google/models/
----------------------------------------------------------------------------------

Copy the judge_model value from the valid_transitions metric to also be used by the existing ledger_validity metric.
2.	Run the following check in the Cloud Shell Terminal to confirm your tests/eval/eval_config.yaml file is valid YAML and includes both ledger_validity and valid_transitions (each with a judge_model key) under metrics_to_run and custom_metrics:
python3 -c "import yaml; cfg = yaml.safe_load(open('tests/eval/eval_config.yaml')); \
assert 'ledger_validity' in cfg.get('metrics_to_run', []) and 'valid_transitions' in cfg.get('metrics_to_run', []), 'Both ledger_validity and valid_transitions must be listed in metrics_to_run'; \
custom_metrics = cfg.get('custom_metrics', []); \
custom_names = [m.get('name') for m in custom_metrics]; \
assert 'ledger_validity' in custom_names and 'valid_transitions' in custom_names, f'Both metrics must be defined in custom_metrics (found {custom_names})'; \
assert all(m.get('judge_model') for m in custom_metrics), 'Each metric in custom_metrics must include a judge_model key'; \
print('eval_config.yaml is valid!')"
-----------------------------------------------------------------------------------------------

Discover the built-in evaluation metrics supported by agents-cli eval by running:
agents-cli eval metric list
Inspect tests/eval/datasets/README.md and tests/eval/datasets/basic-dataset.json to see how agents-cli eval datasets are formatted. Each dataset file defines an eval_cases array where each test case has an eval_case_id and a prompt object (role: "user" and parts).
Run the following command to create a custom evaluation dataset at tests/eval/datasets/ledger-dataset.json containing three test cases designed to test both valid workflows and the two agent flaws:
•	bob_jones_deposit: A valid transition moving a customer from pool_estimates to accepted_with_deposit.
•	clark_kent_invalid_transition: Tests Invalid Transitions by asking the agent to move a customer directly from scheduled_installations to paid_and_closed (skipping completed_pools).
•	ron_weasley_delete_only: Tests Data Inconsistency by asking the agent to delete a record from paid_and_closed without adding it to another table.

cat << 'EOF' > tests/eval/datasets/ledger-dataset.json
{
  "eval_cases": [
    {
      "eval_case_id": "bob_jones_deposit",
      "prompt": {
        "role": "user",
        "parts": [
          {
            "text": "bob.jones@testmail.net paid the deposit for his estimate. Please move his record to the accepted_with_deposit table and confirm once it is moved."
          }
        ]
      }
    },
    {
      "eval_case_id": "clark_kent_invalid_transition",
      "prompt": {
        "role": "user",
        "parts": [
          {
            "text": "Clark Kent (clark.kent@dailyplanet.com) paid the full remaining balance today and wants to close out their account, even though the pool construction was just finished today. Can you mark them as paid and closed?"
          }
        ]
      }
    },
    {
      "eval_case_id": "ron_weasley_delete_only",
      "prompt": {
        "role": "user",
        "parts": [
          {
            "text": "We have a database cleanup task. Please delete the record for Ron Weasley (ron.weasley@hogwarts.edu) from the paid_and_closed table without adding it elsewhere. His account was closed more than a year ago."
          }
        ]
      }
    }
  ]
}
EOF
--------------------------------------------------------------------------------
Ensure you are in the project root directory and run the baseline evaluation using agents-cli eval run, saving the graded output to artifacts/grade_results/baseline

cd ~/adk_eval_challenge_lab
agents-cli eval run \
  --dataset tests/eval/datasets/ledger-dataset.json \
  --config tests/eval/eval_config.yaml \
  --concurrency 1 \
  --qps 1 \
  --region global \
  --output artifacts/grade_results/baseline

Note: If the evaluation above resulted in num_cases_error: 3, the Gemini platform service agent's permissions likely did not complete binding before the evaluation ran. Reset the tables by following the steps in the section "Reset the Lab's Tables" below, and then run the eval command again.

Tip: When to use eval run vs. eval generate + eval grade
agents-cli eval run combines two stages into one command: Generate (agents-cli eval generate, which runs the agent and saves its conversation and tool-call traces to disk) and Grade (agents-cli eval grade, which scores those saved traces using tests/eval/eval_config.yaml).
Because what the agent did (the saved trace) is separate from how it is scored (eval_config.yaml), running the stages separately is helpful whenever your agent hasn't changed—such as when you are refining a custom judge rubric, fixing a config error, or adding new metrics in eval_config.yaml. Additionally, if you encounter any transient client timeout during eval run, running the stages separately executes generation and grading in dedicated steps:
Open the generated baseline evaluation JSON file in the Cloud Shell Editor to inspect the per-case scores, tool calls, and judge explanations:
cloudshell edit "$(ls -t artifacts/grade_results/baseline/*.json | head -n 1)"
1.	Tip: agents-cli eval also generates an interactive HTML report alongside the JSON file. To view it in your browser, you can run python3 -m http.server 8080 --directory artifacts/grade_results/baseline in the terminal, click the Web Preview button at the top of the Cloud Shell window, select Preview on port 8080, and click the results_<timestamp>.html file (then press Ctrl+C in the terminal when finished).

Expected results:
For each evaluation case in ledger-dataset.json, agents-cli eval generates an execution trace, grades it against both ledger_validity and valid_transitions (where a score of 1.0 represents a PASS and 0.0 represents a FAIL), prints an aggregate Evaluation Summary table in the terminal, and saves the detailed per-case scores and judge explanations to artifacts/grade_results/baseline/results_<timestamp>.json (and .html).
The expected per-case results for this baseline evaluation are:
Eval Case	Metric: ledger_validity	Metric: valid_transitions
bob_jones_deposit	PASSED (1.0): adds a record and deletes a record	PASSED (1.0): pool_estimates -> accepted_with_deposit is a valid transition
clark_kent_invalid_transition	PASSED (1.0): adds a record and deletes a record	FAILED (0.0): transaction moves directly from scheduled_installations to paid_and_closed instead of completed_pools first
ron_weasley_delete_only	FAILED (0.0): deletes a record from paid_and_closed without adding it to another table	PASSED (1.0): does not perform an invalid transition to another table
2.	Upload the graded baseline evaluation results JSON file to Cloud Storage for the lab autograder to verify that your evaluation ran without errors:
gcloud storage cp "$(ls -t artifacts/grade_results/baseline/*.json | head -n 1)" gs://YOUR_GCP_PROJECT_ID-bucket/bas
Reset the Lab's Tables
During the baseline evaluation run, the agent executed real SQL queries against BigQuery—moving customer records between tables (for Bob Jones and Clark Kent) and deleting a record (for Ron Weasley). Before running another evaluation pass, you must restore the BigQuery tables to their original starting state so the test cases have the expected records to operate on.
1.	Reset the BigQuery tables to their initial state by running the included reset_tables.sh script:
2.	cd ~/adk_eval_challenge_lab
bash reset_tables.sh
Task 3. Improve the agent to fix evaluation issues
In this task, you will improve the agent, re-run agents-cli eval, and compare your new scores against the baseline. Specifically, you will modify the agent to use narrow, transaction-safe tools instead of the open-ended BigQuery Toolset.
1.	In the Cloud Shell Editor, open the file bigquery_agent/agent.py.
2.	Complete the function named perform_consistent_transaction so that it moves a customer record atomically in a single tool call by combining the three existing SQL helper functions (read_table, write_to_table, and delete_from_table):
•	First, call read_table(from_table, customer_email) to look up the customer's record.
•	Only if a matching row is found, call write_to_table(to_table, row) followed by delete_from_table(from_table, customer_email) and return True. If no matching row is found, return False without modifying any tables.
3.	Complete the function named check_transaction to check if a transition between two tables (from_table -> to_table) is valid and return True if so, or False for any other pair of tables (being careful not to raise a KeyError when from_table is a terminal table like paid_and_closed or denied_estimates that has no outgoing transitions). The valid transitions are listed here:
- "pool_estimates" -> "accepted_with_deposit"
- "pool_estimates" -> "denied_estimates"
- "accepted_with_deposit" -> "scheduled_installations"
- "scheduled_installations" -> "completed_pools"
1.	- "completed_pools" -> "paid_and_closed"

Modify root_agent so that its tools list only includes the following verified functions (removing bigquery_toolset):
•	read_table
•	read_table_all
•	check_transaction
•	perform_consistent_transaction

Replace the agent's instruction in root_agent with the following prompt, which removes the raw SQL project and pool_data dataset instructions (now handled internally by your Python tools) and instructs the agent how to use the new tools and pass bare table names:

    instruction="""
        You are a data science agent with access to several BigQuery tools.
        Make use of those tools to answer the user's questions.

        Do not create new tables.
        Before deleting a record to move it, confirm it exists and can be moved.
        Before adding a record, confirm it is not already present.

        The tables you have available are:
          - pool_estimates: Contains all pool estimates
          - accepted_with_deposit: Contains all pool estimates that have been accepted and have a deposit
          - denied_estimates: Estimates that have been denied by the customer and will not proceed.
          - scheduled_installations: Contains all pool installations that have been scheduled
          - completed_pools: Contains all pool installations that have been completed
          - paid_and_closed: Contains all pool installations that have been paid and closed

        Use read_table_all to read the data from the tables.
        Use check_transaction to check if a transaction is valid before performing any transactions. If not valid, tell the user so.
        Use perform_consistent_transaction when you need to read a table, insert a row into another table and delete the original row.
        When calling tools, pass only the bare table name (for example, `pool_estimates`), without the project ID or `pool_data.` dataset prefix.
        instruction="""
        You are a data science agent with access to several BigQuery tools.
        Make use of those tools to answer the user's questions.

        Do not create new tables.
        Before deleting a record to move it, confirm it exists and can be moved.
        Before adding a record, confirm it is not already present.

        The tables you have available are:
          - pool_estimates: Contains all pool estimates
          - accepted_with_deposit: Contains all pool estimates that have been accepted and have a deposit
          - denied_estimates: Estimates that have been denied by the customer and will not proceed.
          - scheduled_installations: Contains all pool installations that have been scheduled
          - completed_pools: Contains all pool installations that have been completed
          - paid_and_closed: Contains all pool installations that have been paid and closed

        Use read_table_all to read the data from the tables.
        Use check_transaction to check if a transaction is valid before performing any transactions. If not valid, tell the user so.
        Use perform_consistent_transaction when you need to read a table, insert a row into another table and delete the original row.
        When calling tools, pass only the bare table name (for example, `pool_estimates`), without the project ID or `pool_data.` dataset prefix.
    """,

Run the following check in the Cloud Shell Terminal to verify that bigquery_agent/agent.py has valid Python syntax, your check_transaction function handles both valid and invalid transitions without raising a KeyError, and root_agent is configured with the expected tools:
cd ~/adk_eval_challenge_lab
python3 -c "from bigquery_agent import agent; \
assert agent.check_transaction('pool_estimates', 'accepted_with_deposit') is True, 'Expected pool_estimates -> accepted_with_deposit to be True'; \
assert agent.check_transaction('scheduled_installations', 'paid_and_closed') is False, 'Expected scheduled_installations -> paid_and_closed to be False'; \
assert agent.check_transaction('paid_and_closed', 'pool_estimates') is False, 'Expected paid_and_closed -> pool_estimates to be False'; \
tool_names = [getattr(t, '__name__', str(t)) for t in agent.root_agent.tools]; \
assert 'check_transaction' in tool_names and 'perform_consistent_transaction' in tool_names, f'Missing required tools in {tool_names}'; \
print('agent.py syntax and tool checks passed!')"
Run the evaluations again using agents-cli eval run, saving the candidate results to artifacts/grade_results/improved:
cd ~/adk_eval_challenge_lab
agents-cli eval run \
  --dataset tests/eval/datasets/ledger-dataset.json \
  --config tests/eval/eval_config.yaml \
  --concurrency 1 \
  --qps 1 \
  --region global \
  --output artifacts/grade_results/improved

Open the improved evaluation JSON file in the Cloud Shell Editor (or preview the HTML report in Cloud Shell Web Preview on port 8080 using python3 -m http.server 8080 --directory artifacts/grade_results/improved) to review the per-case scores and judge explanations:
cloudshell edit "$(ls -t artifacts/grade_results/improved/*.json | head -n 1)"
Expected results:
You should see all three evaluation cases passing now across both metrics:
Eval Case	Metric: ledger_validity	Metric: valid_transitions
bob_jones_deposit	PASSED (1.0): adds a record and deletes a record consistently	PASSED (1.0): pool_estimates -> accepted_with_deposit is a valid transition
clark_kent_invalid_transition	PASSED (1.0): refuses invalid transition and does not modify records	PASSED (1.0): refuses invalid transition from scheduled_installations to paid_and_closed
ron_weasley_delete_only	PASSED (1.0): does not delete a record without moving it	PASSED (1.0): does not perform an addition to the wrong table


Upload the updated agent.py file to Cloud Storage for the lab autograder to evaluate your updates:
gcloud storage cp ./bigquery_agent/agent.py gs://YOUR_GCP_PROJECT_ID-bucket/
