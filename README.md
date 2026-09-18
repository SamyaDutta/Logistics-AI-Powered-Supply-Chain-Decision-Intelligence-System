# Cold-Chain Logistics FDE

AI-powered supply-chain decision intelligence for cold-chain fleet telemetry, corridor conditions, and compliance operations.

## Live Application

Open the deployed Streamlit application:

**[Cold-Chain Logistics FDE Command Center](https://logistics-ai-powered-supply-chain-decision-intelligence-system.streamlit.app/)**

The application runs with Groq as the reasoning provider and provides two operating modes:

- **Dispatch Console**: Ask operational questions in natural language. The agent can query fleet telemetry, retrieve live corridor conditions, and search cold-chain SOPs.
- **Security & Audit Logs**: Authenticate as an administrator and inspect the agent's recorded tool calls and generated operational responses.

## Problem Statement

Cold-chain logistics teams must protect temperature-sensitive cargo while coordinating vehicles, routes, ports, weather conditions, and compliance procedures. The information needed to make a decision is usually fragmented across:

- Legacy operational databases with difficult-to-understand column names
- Live weather and corridor-condition services
- Static incident procedures and compliance documents
- Separate audit and monitoring workflows

This fragmentation slows incident response and makes it difficult for a dispatcher to answer practical questions such as:

- Which vehicles require immediate attention?
- Is a temperature anomaly connected to route or weather conditions?
- What corrective action does the relevant SOP require?
- Which data sources and tools informed the recommendation?

## Solution

Cold-Chain Logistics FDE provides a decision-intelligence layer over those data sources. Instead of requiring users to write SQL, call APIs, or search policy documents manually, the dispatcher can ask a natural-language question in the Dispatch Console.

The system uses an agent workflow to:

1. Translate the operational request into a structured reasoning task.
2. Query a curated SQL semantic view rather than exposing raw legacy tables directly.
3. Retrieve current corridor conditions when location context is available.
4. Search indexed cold-chain procedures for applicable thresholds and mitigations.
5. Combine the evidence into a concise operational response.
6. Record tool calls, inputs, outputs, and final responses for authorized audit review.

The result is faster incident triage, clearer access to compliance guidance, and traceable decisions without removing human operational control.

## Key Capabilities

- Natural-language fleet telemetry investigation
- Highest-risk vehicle and shipment identification
- Temperature and delay-risk analysis
- Live weather and corridor-condition lookup
- Semantic retrieval over Markdown, text, PDF, CSV, and Excel policy assets
- Groq-powered reasoning with LangGraph tool routing
- Read-only SQL access for agent telemetry queries
- Dedicated semantic view that translates legacy column names into operational terms
- Session-aware conversations in the Streamlit interface
- Audit logging of agent activity and tool execution
- Separate administrator workflow for reviewing audit records

## Technology Stack

### User Interface and Application

- **Python 3.12**: Application runtime
- **Streamlit**: Interactive web dashboard and deployment target
- **Pandas**: Tabular data loading and audit-log display
- **SQLAlchemy**: Database engine and query execution layer
- **pyodbc**: Microsoft SQL Server connectivity through ODBC Driver 18

### AI and Orchestration

- **Groq API**: Hosted reasoning model provider
- **`openai/gpt-oss-120b`**: Configured Groq model
- **LangChain**: Chat model, tools, embeddings, and vector-store integrations
- **LangGraph**: Stateful reasoner and tool-execution graph
- **LangGraph `MemorySaver`**: Conversation checkpointing by session thread
- **Hugging Face Sentence Transformers**: Local embedding generation using `BAAI/bge-m3`

### Data and Retrieval

- **Microsoft SQL Server 2022**: Legacy telemetry and audit-log store
- **Docker**: Local SQL Server runtime container
- **Pinecone**: Vector index for compliance and SOP retrieval
- **Open-Meteo**: Live corridor weather data
- **Markdown, TXT, PDF, CSV, and XLSX**: Supported policy and source formats

### Deployment and Operations

- **GitHub**: Source-code repository
- **Streamlit Community Cloud**: Free hosted application deployment
- **Streamlit Secrets**: Cloud API keys and database configuration
- **PowerShell**: Local Windows development workflow

## Data Flow

### Telemetry Flow

```text
CSV source data
	-> ingest_legacy_data.py
	-> SQL Server dbo.TBL_SC_FLEET_HIST_RAW
	-> FDE_VIEWS.VW_ACTIVE_FLEET
	-> read-only agent SQL tool
	-> Dispatch Console response
```

### Compliance Retrieval Flow

```text
Policy files
	-> ingest_sop_pinecone.py
	-> document parsing and chunking
	-> local BGE-M3 embeddings
	-> Pinecone fde-sop-index-local
	-> compliance retrieval tool
	-> Dispatch Console response
```

### Agent Execution Flow

```text
User question
	-> Groq reasoner
	-> tool call decision
	-> SQL / weather / SOP tool
	-> tool result
	-> Groq synthesis
	-> response and audit record
```

## What This Project Does

The system combines structured data, live external data, and policy documents in one agent workflow:

1. A user submits an operational question in Streamlit.
2. The Groq-powered LangGraph reasoner determines which tools are needed.
3. The agent can call the SQL telemetry view, the corridor weather API, and the Pinecone SOP index.
4. Results are returned to the user as an operational response.
5. Tool inputs, tool outputs, and final responses are written to the audit log for authorized review.

## Architecture

```text
Streamlit UI
	|
	v
LangGraph FDE Orchestrator
	|
	+--> Groq: openai/gpt-oss-120b
	|
	+--> SQL Server: FDE_VIEWS.VW_ACTIVE_FLEET
	|
	+--> Open-Meteo: live corridor conditions
	|
	+--> Pinecone: compliance SOP retrieval
	|
	+--> SQL Server: FDE_VIEWS.AgentAuditLog
```

### Main Components

| Component | Location | Responsibility |
| --- | --- | --- |
| Streamlit dashboard | `src/ui.py` | Dispatch and audit-log user interface |
| LangGraph orchestrator | `src/orchestrator.py` | Reasoning loop and tool routing |
| Agent tools | `src/agent_tools.py` | SQL, weather, and SOP retrieval tools |
| Legacy data loader | `scripts/ingest_legacy_data.py` | Loads the CSV dataset into SQL Server |
| SOP ingestion pipeline | `scripts/ingest_sop_pinecone.py` | Chunks policy files and indexes them in Pinecone |
| Database security script | `scripts/setup_security_and_view.sql` | Creates the semantic view, read-only login, and audit table |
| System prompt | `src/prompts/system_prompt.txt` | Defines the agent's operating behavior |

## Dispatch Console Examples

Select **Dispatch Console** in the sidebar and paste any of the following into the chat box.

### Fleet telemetry

```text
Show the latest 5 fleet telemetry records with Timestamp, Latitude, Longitude, Current_Temperature_C, Risk_Classification, and Delay_Probability.
```

### Highest-risk vehicles

```text
Identify the 10 highest-risk fleet records. Return Timestamp, location, temperature, risk classification, delay probability, port congestion, and route risk. Sort by Route_Risk_Index descending.
```

### Temperature breaches

```text
Find fleet records where Current_Temperature_C is outside a safe cold-chain range. Show the affected coordinates, risk classification, and delay probability, then explain the operational concern.
```

### Corridor conditions

```text
Check the current weather and corridor risk for latitude 33.77 and longitude -118.19.
```

### Compliance SOP retrieval

```text
What temperature limits and corrective actions apply to fresh perishables during a cold-chain breach?
```

### Full multi-tool workflow

```text
Identify the 5 highest-risk fleet records, check corridor conditions for the first record's coordinates, and explain the recommended operational response using the cold-chain compliance SOP.
```

The full workflow should exercise the SQL telemetry tool, corridor weather API, Pinecone retrieval, and Groq reasoning in a single request.

## Security & Audit Logs Examples

Select **Security & Audit Logs** in the sidebar. Enter the configured administrator credentials and click **Authenticate & Load Logs**.

The audit view is designed to show:

- Session token
- Graph node that ran
- Tool that was triggered
- Generated tool arguments
- Raw tool output
- Final model response
- Execution timestamp

The audit page does not use chat prompts. Use the following actions to generate records first in **Dispatch Console**, then return to **Security & Audit Logs**:

1. Run the fleet telemetry example.
2. Run the corridor conditions example.
3. Run the compliance SOP example.
4. Return to **Security & Audit Logs** and authenticate.
5. Review the recorded reasoner and tool activity.

For an audit-focused operational review, run this in Dispatch Console first:

```text
Explain why the highest-risk fleet record needs attention, identify every data source you used, and provide a concise recommended action.
```

## Local Installation

### Prerequisites

- Python 3.12
- Docker Desktop
- SQL Server ODBC Driver 18
- A Groq API key
- A Pinecone API key

The project was run successfully with the external virtual environment at `C:\venvs\coldchain`, which keeps dependencies outside OneDrive.

### Create or activate the environment

```powershell
C:\venvs\coldchain\Scripts\Activate.ps1
```

Install dependencies:

```powershell
python -m pip install -r requirements.txt
```

### Configure local secrets

Create a `.env` file in the project root. Never commit it or publish its values.

```dotenv
SQL_SERVER_HOST=localhost
SQL_SERVER_PORT=1433
SQL_ADMIN_USER=sa
SQL_ADMIN_PASSWORD=your-admin-password
SQL_AGENT_USER=USR_FDE_RO
SQL_AGENT_PASSWORD=your-agent-password
PINECONE_API_KEY=your-pinecone-key
GROQ_API_KEY=your-groq-key
Embeddings_model=LOCAL
Local_Embedding_Model=BAAI/bge-m3
Agent_llm=GROQ
```

### Start SQL Server

The local development setup uses the Docker container `legacy-mssql` on port `1433`.

```powershell
docker ps
```

Load the legacy CSV data:

```powershell
python scripts/ingest_legacy_data.py
```

Apply the security and semantic-layer SQL script in the `master` database:

```text
scripts/setup_security_and_view.sql
```

Index the compliance documents in Pinecone:

```powershell
python scripts/ingest_sop_pinecone.py
```

Start the dashboard:

```powershell
streamlit run src/ui.py
```

Then open `http://localhost:8501`.

## Streamlit Community Cloud Deployment

1. Push the project to GitHub.
2. Create a new app at [Streamlit Community Cloud](https://share.streamlit.io/).
3. Select the repository and `main` branch.
4. Set the main file to `src/ui.py`.
5. Select Python `3.12` in the advanced settings.
6. Add deployment secrets in **Advanced settings > Secrets**.

Example secret names:

```toml
GROQ_API_KEY = "your-groq-key"
PINECONE_API_KEY = "your-pinecone-key"
Agent_llm = "GROQ"
Embeddings_model = "LOCAL"
Local_Embedding_Model = "BAAI/bge-m3"
SQL_SERVER_HOST = "your-public-sql-host"
SQL_SERVER_PORT = "1433"
SQL_AGENT_USER = "USR_FDE_RO"
SQL_AGENT_PASSWORD = "your-agent-password"
SQL_ADMIN_USER = "sa"
SQL_ADMIN_PASSWORD = "your-admin-password"
```

Do not set `SQL_SERVER_HOST` to `localhost` in Streamlit Cloud. In the cloud, `localhost` refers to the Streamlit app container, not the computer running Docker. Use a secured, publicly reachable SQL Server or a managed database instead.

## Data and Policy Inputs

- `data/raw/dynamic_supply_chain_logistics_dataset.csv`: source fleet telemetry dataset
- `data/policy/Cold_Chain_Incident_SOP_v2.md`: compliance policy used for retrieval
- `data/source/data.txt`: additional source material

The Pinecone ingestion script supports Markdown, text, PDF, CSV, and Excel policy files.

## Security Notes

- Keep `.env` and Streamlit secrets private.
- Use the read-only `USR_FDE_RO` login for agent telemetry queries.
- The telemetry tool accepts only `SELECT` statements.
- Use administrator credentials only for the audit-log screen.
- Rotate API keys and database passwords immediately if they are exposed.
- Do not expose a local SQL Server directly to the internet without network controls, encryption, and access restrictions.

## Troubleshooting

### Pinecone Unauthorized error

Confirm that `PINECONE_API_KEY` belongs to the Pinecone project containing `fde-sop-index-local`. Update the Streamlit Cloud secret and reboot the app.

### SQL connection error in Streamlit Cloud

Confirm that `SQL_SERVER_HOST` is a public database endpoint. A local Docker address such as `localhost:1433` is reachable only from the local machine.

### Missing Python package

Use Python 3.12 and reinstall from `requirements.txt` in the active environment.

## License

See [LICENSE](LICENSE).

