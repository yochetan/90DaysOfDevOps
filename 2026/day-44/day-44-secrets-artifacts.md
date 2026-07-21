Task 1: GitHub Secrets

1) Go to your repo → Settings → Secrets and Variables → Actions

        yeah did.

2) Create a secret called MY_SECRET_MESSAGE

        created.

3) Create a workflow that reads it and prints: The secret is set: true (never print the actual value)

        run: |
                          if [ -n "$MY_SECRET" ]; then
                            echo "The secret is set: true"
                          else
                            echo "The secret is set: false"
                          fi

4) Try to print ${{ secrets.MY_SECRET_MESSAGE }} directly — what does GitHub show?

        - name: Try to print the secret (for learning only)
                        run: |
                          echo "Secret value: ${{ secrets.MY_SECRET_MESSAGE }}"

It printed this:

        Secret value: ***

`secrets-demo.yml`

        name: Secrets Demo
        
        on:
          workflow_dispatch: 
        
        jobs:
          test-secret:
            runs-on: ubuntu-latest
        
            steps:
              - name: Check if secret exists
                env:
                  MY_SECRET: ${{ secrets.MY_SECRET_MESSAGE }}
                run: |
                  if [ -n "$MY_SECRET" ]; then
                    echo "The secret is set: true"
                  else
                    echo "The secret is set: false"
                  fi
        
              - name: Try to print the secret (for learning only)
                run: |
                  echo "Secret value: ${{ secrets.MY_SECRET_MESSAGE }}"

Write in your notes: Why should you never print secrets in CI logs?

a) CI logs may be visible to collaborators or anyone with repository access.

b) Secrets include API keys, passwords, tokens, cloud credentials, SSH keys, etc.

c) If exposed, an attacker could:

- Access your cloud infrastructure.
- Push malicious code.
- Deploy or delete applications.
- Read or modify databases.
- Impersonate your services.

d) Even though GitHub masks repository secrets, you should never intentionally print them because:

- Masking is not guaranteed for every transformed or partial value.
- Secrets copied into other variables or external outputs may not always be protected.
- Good security practice is to use secrets only where needed, not display them.


Task 2: Use Secrets as Environment Variables

1) Pass a secret to a step as an environment variable

2) Use it in a shell command without ever hardcoding it

3) Add DOCKER_USERNAME and DOCKER_TOKEN as secrets (you'll need these on Day 45)

