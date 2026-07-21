Task 1: GitHub Secrets

1) Go to your repo → Settings → Secrets and Variables → Actions

        yeah did.

2) Create a secret called MY_SECRET_MESSAGE

3) Create a workflow that reads it and prints: The secret is set: true (never print the actual value)

4) Try to print ${{ secrets.MY_SECRET_MESSAGE }} directly — what does GitHub show?

Write in your notes: Why should you never print secrets in CI logs?
