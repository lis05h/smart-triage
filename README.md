# S.M.A.R.T. Drive Triage

An educational generative AI application developed for the Introduction to Generative AI course.

## Project overview

I developed S.M.A.R.T. Drive Triage to make storage-drive reports easier to interpret. S.M.A.R.T. telemetry contains technical measurements, normalized values, and raw attributes whose meanings may depend on the manufacturer.

My application extracts evidence from a supplied report, assigns a cautious triage category, and presents a readable assessment. I designed it to interpret report text—not to predict failure dates, scan drives, or provide a certified diagnosis.

This repository contains my completed individual course project.

## How the application works

1. A report is entered into the Streamlit interface.
2. Python checks for empty input before making an API request.
3. A Groq request asks the model to extract evidence, select an assessment category, and return JSON.
4. Python validates the response structure, field types, and assessment label.
5. The interface displays:
   - Triage assessment.
   - A predefined category-specific summary.
   - An evidence table containing reported values and generated interpretations.
   - Predefined recommended actions.
   - Important Notes.

I separated the generated analysis from the predefined summaries and actions to control presentation, avoid executable diagnostic advice, and prioritize data protection when concerning findings are reported.

The model selects the assessment category. JSON validation checks structure; it does not independently establish whether every interpretation or classification is correct.

## Technology

- Python.
- Streamlit `1.66.1`.
- Groq Python library `1.7.0`.
- Groq model: `openai/gpt-oss-20b`.

I used Python 3.13 during development. The application is not restricted to Python 3.13; another Python version may be used when compatible with the pinned packages and their dependencies.

The application requires internet access and a private Groq API key. It does not require a database, a locally installed language model, or a separate backend service.

## Prompt engineering

I used two principal prompt techniques:

- Role assignment: I instructed the model to act as a cautious storage reliability engineer.
- Few-shot prompting: I included illustrative examples covering missing telemetry, concerning SATA sector counts, unexplained raw encoding, no-warning NVMe reports, warning NVMe reports, and unrelated input.

I also defined a structured JSON response and explicit evidence-handling rules. These instructions address:

- Preserving reported values, including leading zeros.
- Distinguishing raw values from normalized values and thresholds.
- Avoiding invented measurements.
- Treating missing information as unknown rather than zero.
- Acknowledging vendor-specific encoding.
- Avoiding unsupported deterioration, wear, and failure-timing claims.
- Keeping potentially overlapping counts separate.
- Interpreting NVMe Percentage Used as estimated endurance consumed rather than occupied storage capacity.
- Describing operating-hour and power-cycle counters neutrally.
- Keeping warning-temperature and critical-temperature counters distinct.
- Restricting uncertainty notes to information relevant to the supplied report.
- Excluding commands, device paths, and diagnostic instructions from generated content.

I refined these instructions through recorded tests and retained unsuccessful outputs as evidence of the development process.

## Generation settings

The sidebar provides:

| Setting | Default |
|---|---:|
| Temperature | `0.2` |
| Maximum output tokens | `2048` |

These are request-time generation settings, not model training or fine-tuning.

I used the default configuration for the recorded final regression set. I did not conduct a controlled comparison of different parameter values.

## Installation on Windows

### 1. Obtain the project files

Download and extract the repository ZIP, or clone the repository.

Open the folder containing `app.py` and `requirements.txt` in a terminal or editor.

### 2. Check Python

List the available Python installations:

```powershell
py -0p
```

Check the default Python version:

```powershell
py --version
```

Use a Python installation compatible with the pinned dependencies. Python 3.13 is the recorded development environment, not an exclusive requirement.

### 3. Create a virtual environment

Using a compatible default Python installation:

```powershell
py -m venv .venv
```

To select a specific installed version, use its launcher selector. For example:

```powershell
py -3.13 -m venv .venv
```

### 4. Install dependencies

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

The direct dependencies are pinned in `requirements.txt`:

```text
streamlit==1.66.1
groq==1.7.0
```

These instructions invoke the virtual environment’s Python directly, so PowerShell environment activation is unnecessary.

### 5. Configure the private API key

Create a `.streamlit` folder inside the project folder.

Inside it, create `secrets.toml`:

```toml
GROQ_API_KEY = "YOUR_PRIVATE_GROQ_KEY"
```

Replace the placeholder with a private Groq API key.

Do not commit this file, include the key in screenshots, or place credentials in documentation.

The repository’s `.gitignore` should contain:

```gitignore
.venv/
.streamlit/secrets.toml
__pycache__/
```

For manual uploads, check the selected files explicitly. Do not rely on `.gitignore` to protect a manually selected secrets file.

### 6. Launch the application

```powershell
.\.venv\Scripts\python.exe -m streamlit run app.py
```

Open the local address shown in the terminal.

Keep the terminal running while using the application. Stop it with `Ctrl+C`.

## Using the application

1. Remove serial numbers and other identifying information from the report.
2. Paste the report into the S.M.A.R.T. report text box.
3. Start with temperature `0.2` and maximum output tokens `2048`.
4. Click Analyze.
5. Read the assessment, evidence, recommended actions, and Important Notes together.

The available assessment categories are:

- No obvious warning in supplied data.
- Monitor.
- Urgent attention.
- Insufficient information.
- Invalid input.

“Urgent attention” identifies findings that warrant prompt data protection and follow-up. It does not predict when a drive will fail.

“No obvious warning in supplied data” is restricted to the supplied measurements. It is not a guarantee of overall reliability.

## Data handling and application boundaries

The entered report text is sent to Groq for analysis. Identifying details should be removed before submission.

The application:

- Processes pasted text.
- Does not scan drives.
- Does not execute diagnostic commands.
- Does not change hardware settings.
- Does not maintain a report-history database.
- Does not provide a built-in report-export feature.

The screenshots and PDFs used in the testing record are documentation evidence, not application export functionality.

## Input and response handling

I implemented handling for:

- Empty input.
- Missing or empty API credentials.
- Authentication failures.
- Connection failures.
- Rate limits.
- API failures.
- Invalid JSON or unexpected response structure.
- Unrecognized assessment labels.
- Output truncation.

When the output-token limit is reached, the application displays a warning rather than an incomplete assessment.

Implemented handling and experimentally verified behavior are distinguished in the testing documentation. I do not describe every implemented handler as independently tested.

## Testing and Results

I tested synthetic SATA HDD and NVMe SSD reports, missing-telemetry input, and unrelated text.

My development record includes:

- Initial unsuccessful outputs.
- Prompt revisions and retests.
- Repeated runs with unchanged input.
- Changed-value tests.
- Counter-interpretation checks.
- Recommendation-order checks.
- Interface-heading checks.
- A final six-case regression set.

The final regression set covered:

| Test case | Observed assessment |
|---|---|
| Concerning SATA sector counters | Urgent attention |
| Unexplained vendor-specific raw value | Insufficient information |
| No-warning NVMe report | No obvious warning in supplied data |
| Warning NVMe report with spare below threshold | Urgent attention |
| Missing telemetry | Insufficient information |
| Unrelated request | Invalid input |

Each case met its predefined acceptance checks in the recorded final output.

These results represent a targeted manual regression set, not a general diagnostic-accuracy score. Some test cases resemble the prompt examples. Earlier repeated runs used preceding prompt revisions and are not repeated trials of the final prompt.

The testing appendix includes input definitions, individual test records, observed output excerpts, and supporting captures. Historical observations without a confirmed original-run mapping are identified separately rather than assigned invented details.

## Assessment Scope

I designed the application to interpret the supplied report as a snapshot. A single report does not establish a deterioration trend or predict failure timing.

Raw attributes may require manufacturer-specific documentation. I instructed the model to preserve unexplained values rather than invent their meanings.

The application’s validator checks response structure and field types. It does not independently verify every measurement, interpretation, category, or generated statement against a manufacturer database.

Predefined summaries and recommended actions control those sections of the interface, but their selection still depends on the model’s assessment category. Generated evidence interpretations and notes should be read alongside the supplied values.

An overall `PASSED` result does not establish that every attribute is normal or guarantee drive reliability.

## Project materials

| File or document | Purpose |
|---|---|
| `app.py` | Application, prompt, API integration, validation, and display logic |
| `requirements.txt` | Pinned direct dependencies |
| `.gitignore` | Local-environment and credential exclusions |
| `README.md` | Project overview, installation, and usage |
| Project report PDF | Purpose, implementation, prompt techniques, design decisions, and testing summary |
| Testing appendix | Detailed experiment history and supporting evidence |

I completed this application as an individual project for the Introduction to Generative AI course. The accompanying report and testing appendix document the submitted version and the work I performed.
