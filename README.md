# S.M.A.R.T. Drive Triage

An educational generative AI application for the Introduction to Generative AI course.

## Problem and purpose

Pasted S.M.A.R.T. telemetry can be difficult to interpret, particularly when raw attributes have vendor-specific meanings. This application extracts reported evidence, assigns a cautious triage category, and presents a plain-language report. It is not a failure-prediction system or a certified diagnostic tool.

## How it works

1. The user pastes a text report into a Streamlit page.
2. Python rejects empty input before making an API call.
3. One Groq request asks the model to extract evidence, categorize concerns, and describe limitations as JSON.
4. Python validates the response structure and assessment label.
5. The application displays the extracted evidence and AI-generated interpretations and limitations.
6. Python supplies predefined summaries and recommended actions for the selected category.

JSON validation checks structure, not whether the model's interpretation is correct. There is no independent numeric parser or vendor database. The model selects the category; predefined text does not independently verify that selection.

## Technology

- Python 3.13 (tested)
- Streamlit 1.66.1
- Groq Python library 1.7.0
- Groq model: `openai/gpt-oss-20b`

No database, local AI model, Git installation, or separate backend service is required for the browser-upload workflow described below. Internet access and a Groq API key are required for analysis. API availability and account quotas depend on the provider.

## Prompt engineering

- Role/persona: a cautious storage reliability engineer.
- Few-shot prompting: examples of missing telemetry, concerning sector counts, unknown raw encoding, and non-report input.
- Structured output: the model returns JSON for validation and display.
- Evidence constraints: preserve values, distinguish raw from normalized values, avoid invented measurements, and acknowledge uncertain vendor-specific encoding.

The sidebar exposes temperature and maximum completion tokens. These are request-time generation settings, not model training or fine-tuning. They may affect response variability and length; low temperature does not guarantee correctness. The token budget may also be consumed by model reasoning, so a short budget can leave little visible output.

## Setup on Windows 11

### 1. Get the project files

Sign in to GitHub and open this repository. Select **Code > Download ZIP**, then extract it. Choose the extracted folder containing `app.py` and `requirements.txt` as the project folder. Open that folder in VS Code using **File > Open Folder**.

Alternatively, reuse the existing `C:\Users\User\smart-triage` folder if it already contains your latest files. Do not overwrite newer local work with an older download.

### 2. Open a terminal in the project folder

In VS Code, select **Terminal > New Terminal**. Confirm the terminal path is the folder containing `app.py`.

Check installed Python versions:

```powershell
py -0p
```

These instructions use Python 3.13. If it is absent, arrange a permitted per-user Python installation rather than assuming Python 3.9 can run the pinned packages.

### 3. Create the environment and install packages

Run these commands once for each new project folder on a computer:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Do not copy `.venv` between computers. No PowerShell activation or execution-policy change is needed because the commands invoke the environment's Python directly.

### 4. Configure the private API key

Create a `.streamlit` folder inside the project folder. Inside it, create `secrets.toml` containing:

```toml
GROQ_API_KEY = "YOUR_PRIVATE_GROQ_KEY"
```

Use your own key, not the placeholder. Never upload this file, include it in screenshots, or paste it into chat. It is plain text on the computer, not encrypted storage. On shared computers, do not leave the key accessible to other people. Keep your recoverable copy in an approved password manager or other private storage, not in this repository.

The project's `.gitignore` should contain:

```gitignore
.venv/
.streamlit/secrets.toml
__pycache__/
```

For manual browser uploads, explicitly select only intended non-secret files. Do not rely on `.gitignore` to protect a manually selected upload.

### 5. Start the application

```powershell
.\.venv\Scripts\python.exe -m streamlit run app.py
```

Open the local address printed in the terminal. Keep that terminal running while using the application. Stop the application with `Ctrl+C`.

## Using the application

1. Remove serial numbers and other identifying information from the report.
2. Paste the report into the text box.
3. Start with temperature `0.2` and maximum output tokens `2048`.
4. Click **Analyze**.
5. Read the assessment, evidence, recommended actions, and limitations together.

The application analyzes pasted text only. It does not scan drives, execute diagnostic commands, or change hardware settings. The submitted text is sent to Groq. The application has no scan-history database or report-export feature.

## Continue on another computer

Before leaving:

1. Save your changes in VS Code.
2. Upload changed non-secret project files to the same locations in the GitHub repository and commit them.
3. Confirm the uploaded files contain the latest changes.
4. Stop the application. Sign out of shared-browser sessions and protect or remove locally stored credentials as appropriate.

On the next computer:

1. Download the latest repository files and open the project folder.
2. Check Python availability.
3. Create `.venv` and install `requirements.txt` if that project folder has no working environment yet.
4. Recreate the private secrets file if needed.
5. Start the application with the command above.

GitHub browser uploads are manual backups, not automatic synchronization. If reusing an existing folder, update the changed source/document files from GitHub without copying environments or exposing secrets.

## Error handling

The application includes handling for empty input, missing or empty API keys, authentication errors, connection failures/timeouts, rate limits, API status errors, invalid structured responses, and output truncation. Implemented handlers should not be described as live-tested unless recorded in `TESTING.md`.

If the response is cut off, increase the output token setting and retry. For a rate limit, wait before trying again. For a connection error, restore internet access and retry. Never share credentials when reporting an error.

## Evaluation and limitations

See `TESTING.md` for manual tests and the prompt-refinement record. Evaluation used a small synthetic SATA HDD test set; it does not establish general diagnostic accuracy or comprehensive HDD, SSD, or NVMe support.

Earlier free-text outputs sometimes included commands and unsupported failure-timing claims despite prompt rules. The revised design uses validated JSON plus predefined summaries and actions. AI-generated evidence interpretations and limitations can still be wrong or omit relevant details. A single snapshot cannot establish a deterioration trend or predict a failure date, and a PASSED result cannot guarantee reliability.

## Project files

- `app.py`: application, prompt, response validation, and display logic.
- `requirements.txt`: pinned direct dependencies.
- `.gitignore`: exclusions for local environments and credentials.
- `README.md`: overview, setup, use, and cross-computer workflow.
- `TESTING.md`: test results, limitations, and refinement history.

The final submission also includes a separate project-documentation PDF. That document explains the implementation and design decisions; it is not merely a browser printout of one analysis.
