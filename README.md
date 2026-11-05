# S.M.A.R.T. Drive Triage

S.M.A.R.T. Drive Triage is a Streamlit web application that helps a user interpret a pasted `smartctl` S.M.A.R.T. report. It uses a generative-AI model through the Groq API to extract reported measurements, identify potential concerns, assign a triage category, and display evidence-based notes.

The application is intended as an AI-assisted first review of drive-health information. It does **not** predict when a drive will fail, guarantee drive reliability, or replace professional data recovery or hardware support.

## GitHub Repository

https://github.com/lis05h/smart-triage

## What the application does

The user pastes a S.M.A.R.T. report into the text box and selects **Analyze**. The application then:

1. Sends the report to the Groq API.
2. Asks the model to return a structured JSON assessment.
3. Validates that the response has the expected structure.
4. Displays the assessment category, evidence, recommended actions, and important notes.

The application supports these assessment categories:

- **No obvious warning in supplied data**
- **Monitor**
- **Urgent attention**
- **Insufficient information**
- **Invalid input**

Recommended actions are supplied by the application rather than generated freely by the model. This reduces the risk of unsafe command-line instructions, device paths, or unsupported diagnostic claims.

## Generative-AI techniques

The application uses several prompt-engineering techniques:

- **Role assignment:** The system prompt assigns the model the role of a cautious storage reliability engineer.
- **Structured output:** The model is instructed to return JSON with fixed keys.
- **Few-shot examples:** The prompt includes examples for missing telemetry, concerning sector values, unknown vendor values, normal NVMe values, failed NVMe values, and invalid input.
- **Explicit constraints:** The prompt limits conclusions to the supplied report, prevents unsupported failure predictions, and separates AI analysis from fixed recommendations.
- **Parameterization:** The user can adjust temperature and maximum output tokens in the sidebar.

## Analysis settings

### Temperature

Temperature controls the randomness of the model’s output. A lower value makes responses more consistent and predictable, while a higher value allows more variation.

S.M.A.R.T. Drive Triage uses a default temperature of **0.2**. This low value supports consistent, cautious, evidence-based interpretation rather than creative variation. The user can adjust it between **0.0 and 1.0**.

### Maximum output tokens

A token is a small unit of text, such as a word, part of a word, or punctuation mark. The maximum output-token setting limits the length of the AI response.

The application uses a default limit of **2,048 output tokens**. This provides enough space for a complete structured assessment while preventing excessively long responses. The user can adjust the limit between **1,024 and 4,096** tokens.

If the response reaches the token limit before completion, the application warns the user and does not display an incomplete assessment.

## Model and API settings

The application uses the Groq-hosted `openai/gpt-oss-20b` model. It requests JSON output using `response_format: json_object` and validates the returned JSON before displaying results.

The Groq client uses a 30-second timeout. Automatic retries are disabled, so the application can show a clear error message instead of silently repeating a failed request.

## Requirements

You need:

- Python 3.10 or newer
- A Groq API key
- An internet connection

## Installation

1. Download or clone this repository.

2. Open a terminal in the project folder.

3. Create and activate a virtual environment.

On Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

On macOS or Linux:

```bash
python3 -m venv venv
source venv/bin/activate
```

4. Install the required packages:

```bash
pip install -r requirements.txt
```

## Groq API key setup

The application reads the API key from Streamlit Secrets. Do **not** put your API key directly in `app.py` or upload it to GitHub.

1. In the project folder, create a folder named `.streamlit`.
2. Inside `.streamlit`, create a file named `secrets.toml`.
3. Add your Groq API key:

```toml
GROQ_API_KEY = "your-groq-api-key-here"
```

4. Save the file.

The `.gitignore` file prevents `secrets.toml` from being uploaded to GitHub.

## Running the application

With the virtual environment active, run:

```bash
streamlit run app.py
```

Streamlit will display a local web address, usually:

```text
http://localhost:8501
```

Open that address in your browser.

## How to use the application

1. Paste a `smartctl` S.M.A.R.T. report into the text area.
2. Remove serial numbers and other identifying information before submitting it.
3. Optionally adjust **Temperature** and **Maximum output tokens** in the sidebar.
4. Select **Analyze**.
5. Review the assessment, evidence, recommended actions, and important notes.

Example input:

```text
smartctl 7.4
=== START OF INFORMATION SECTION ===
Model Number: EXAMPLE NVME SSD
=== START OF SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED
Critical Warning: 0x00
Temperature: 35 Celsius
Available Spare: 100%
Available Spare Threshold: 10%
Percentage Used: 3%
Power Cycles: 150
Power On Hours: 2400
Unsafe Shutdowns: 0
Media and Data Integrity Errors: 0
Error Information Log Entries: 0
```

## Error handling

The application handles common problems without crashing:

- Empty input: asks the user to paste a S.M.A.R.T. report.
- Missing API key: displays setup instructions.
- Invalid API key: asks the user to check the key.
- Rate limit: asks the user to wait before trying again.
- Connection error or timeout: asks the user to try again.
- Incomplete or invalid AI response: displays an error instead of showing incomplete results.
- Token-limit response: warns the user and avoids displaying an incomplete assessment.

## Scope and Considerations

- The application analyzes only the S.M.A.R.T. information supplied in the pasted report.
- A single report is a snapshot and cannot establish a deterioration trend.
- The application cannot predict an exact failure date or remaining drive lifetime.
- A `PASSED` overall-health result does not guarantee that a drive is reliable.
- Vendor-specific raw values may require manufacturer documentation and are not guessed by the application.
- AI assessments can be incorrect and must not be treated as professional hardware diagnostics.

## Documentation

- `SMART_Drive_Triage_Final_Report.pdf` contains the project report.
- `SMART_Drive_Triage_Testing_Appendix_Final.pdf` contains the testing definitions, results, and original output evidence.

## Repository contents

```text
app.py
README.md
requirements.txt
.gitignore
SMART_Drive_Triage_Final_Report.pdf
SMART_Drive_Triage_Testing_Appendix_Final.pdf
```

## Author

**Lis Hadërgjonaj**  
Neptun code: **B5KCCW**  
University of Debrecen — Faculty of Informatics  
Computer Science Engineering BSc  
Introduction to Generative AI — Individual project
