# Setup Guide

## Prerequisites

- An accessible n8n instance.
- The credentials and external services required by the final workflow.
- The workflow JSON export, once added to `workflow/`.

## Import the Workflow

1. Open your n8n workspace.
2. Choose the option to import a workflow from a file.
3. Select the project JSON file from the `workflow/` directory.
4. Review every node and reconnect the required credentials using n8n's credential manager.
5. Configure webhook URLs and any external service settings required by the workflow.
6. Test with synthetic patient data before enabling the workflow.

## Credentials and Configuration

The exact credentials depend on the final workflow export. Configure them inside n8n; do not hardcode secrets in workflow JSON or commit them to GitHub.

## Testing Checklist

- Test a valid intake request using synthetic data.
- Test missing or malformed fields.
- Test appointment-related edge cases supported by the workflow.
- Verify error handling and the response returned to the requester.
- Confirm that logs and external services do not expose sensitive patient information.

## Before Production

- Add the reviewed workflow JSON to `workflow/`.
- Replace this generic guide with the workflow's exact node and credential requirements.
- Review privacy, security, access controls, retention, and applicable healthcare obligations.
- Obtain appropriate human review for urgent or clinical matters.
