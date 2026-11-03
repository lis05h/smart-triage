# Testing and Prompt Refinement

## Test environment

- Windows 11
- Python 3.13
- Streamlit 1.66.1
- Groq Python library 1.7.0
- Model: openai/gpt-oss-20b
- Analysis settings: temperature 0.2, maximum output tokens 2048

## Scope

Tests used synthetic SATA HDD reports and deliberately incomplete or
invalid input. They demonstrate selected application behaviors, not
diagnostic accuracy across all drives, vendors, or report formats.

## Manual test results

| Test | Expected behavior | Observed result |
|---|---|---|
| Concerning sector counts: 24 reallocated, 8 pending, 8 uncorrectable | Urgent attention; preserve values; protect data before diagnostics | Passed on revised application |
| Zero sector counts | No obvious warning; no urgent replacement recommendation | Passed on revised application |
| Unknown raw encoding: Raw_Read_Error_Rate = 987654321 | Insufficient information; do not interpret as a literal failed-read count | Passed after structured-output revision |
| Missing telemetry | Insufficient information; no invented measurements or assumed hardware fault | Passed on revised application |
| Empty input | Display validation message without sending an API request | Passed on revised application |
| Non-report text | Explain that the input is not a SMART report | Passed on an earlier version; final-version retest pending |
| Internet disconnected | Display a friendly connection-error message | Passed on an earlier version; final-version retest pending |

## Prompting techniques

1. Role/persona: the model acts as a cautious storage reliability engineer.
2. Few-shot prompting: examples demonstrate missing telemetry, concerning
   sector counts, ambiguous vendor-specific raw values, and invalid input.

## Refinement process

The initial application generated a free-text report. Testing revealed
unsupported failure-timing claims, unnecessary diagnostic advice, and
command examples despite instructions prohibiting them.

Additional prompt rules improved some responses but did not consistently
prevent these issues.

The revised design requests JSON and validates the response structure.
The model extracts and interprets evidence and selects an assessment.
Python supplies predefined assessment summaries and recommended actions.

This controls those displayed sections without adding another API call.
Evidence interpretations and limitations remain AI-generated and may
still contain mistakes.

## Limitations

- The test set is small and synthetic.
- No real-drive diagnostic accuracy or failure-prediction performance
  has been established.
- No comprehensive SSD, NVMe, or vendor-specific evaluation was performed.
- Structured-response validation checks format, not factual correctness.
- Repeated runs may produce different assessments or interpretations.
- Authentication, rate-limit, malformed-response, and output-truncation
  handling require additional live verification.
- The application is an educational triage aid, not a certified diagnostic
  tool or a guarantee of drive reliability.