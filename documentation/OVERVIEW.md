# Project Overview

## Purpose

The Healthcare Patient Intake & Appointment Agent is an n8n automation project intended to support the intake of patient messages and appointment-related processes.

## Suggested Workflow Documentation

When the final workflow is added, document:

1. **Trigger:** How patient messages or intake requests enter the workflow.
2. **Input validation:** Required fields and handling of incomplete submissions.
3. **Intake processing:** How incoming requests are interpreted and categorized.
4. **Appointment handling:** Any scheduling, availability, or escalation steps implemented by the workflow.
5. **Response:** How the workflow responds to the requester.
6. **Error handling:** How failures and requests requiring human review are managed.

These items are a documentation outline, not confirmation that each step is implemented. Update this file to match the actual n8n workflow.

## Privacy Considerations

- Use synthetic patient details in examples and tests.
- Keep credentials in n8n's credential manager or secure environment configuration.
- Avoid committing protected health information, API keys, tokens, or personal data.
- Require appropriate human review for clinical, urgent, or ambiguous requests.

## Related Files

- [Main README](../README.md)
- [Setup Guide](../setup/SETUP.md)
- [Workflow Directory](../workflow/)
- [Examples Directory](../examples/)
