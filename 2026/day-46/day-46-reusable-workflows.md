Task 1: Understand workflow_call

Before writing any code, research and answer in your notes:

1) What is a reusable workflow?

        A reusable workflow is a GitHub Actions workflow that is designed to be called by other workflows instead of running directly for every event. It lets you write common CI/CD logic once and reuse it across multiple repositories or workflows.

Benefits:

Reduces duplicate YAML code.
Makes workflows easier to maintain.
Ensures consistent CI/CD processes across projects.

Example use cases:

Standard build and test pipeline
Docker image build and push
Security scanning
Deployment workflow
2) What is the workflow_call trigger?

3) How is calling a reusable workflow different from using a regular action (uses:)?

4) Where must a reusable workflow file live?
