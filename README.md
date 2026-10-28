# Healthcare Patient Intake & Appointment Agent

An n8n automation project for organizing patient intake messages and supporting appointment-related workflows.

## Repository Structure

```text
Healthcare-Patient-Intake-Appointment/
├── README.md
├── documentation/
│   └── OVERVIEW.md
├── setup/
│   └── SETUP.md
├── workflow/
│   └── .gitkeep
└── examples/
    └── .gitkeep
```

## Project Components

- **Workflow:** Store the importable n8n workflow JSON in `workflow/`.
- **Documentation:** Project purpose, workflow steps, and design notes.
- **Setup:** Credentials, environment configuration, import steps, and testing instructions.
- **Examples:** Synthetic sample requests and example responses for testing.

## Getting Started

1. Review [the project overview](documentation/OVERVIEW.md).
2. Follow the [setup guide](setup/SETUP.md).
3. Import the workflow JSON from `workflow/` into n8n once it has been added.
4. Configure credentials in n8n; do not commit API keys, access tokens, patient data, or other secrets.

## Privacy and Safety

Use synthetic data for development and testing. Do not store real patient information in this public repository. Review applicable privacy, security, and healthcare requirements before using the workflow with real data.

## Status

Repository structure and starter documentation are in place. Add and validate the final workflow export and project-specific configuration details before deployment.
