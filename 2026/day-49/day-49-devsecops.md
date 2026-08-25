Task 1: Scan Your Docker Image for Vulnerabilities

Your Docker image might use a base image with known security issues. Let's find out.

Add this step to your main branch pipeline (after Docker build, before deploy):

    - name: Scan Docker Image for Vulnerabilities
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'your-username/your-app:latest'
        format: 'table'
        exit-code: '1'
        severity: 'CRITICAL,HIGH'

What this does:

* trivy scans your Docker image for known CVEs (Common Vulnerabilities and Exposures)
* format: 'table' prints a readable table in the logs
* exit-code: '1' means fail the pipeline if CRITICAL or HIGH vulnerabilities are found
* If it passes, your image is clean — proceed to push and deploy

Push and check the Actions tab. Read the scan output.
    
    Success

Verify: Can you see the vulnerability table in the logs? Did it pass or fail?
    
    I got fails in the node_modules
    but I fixed it by writing a multistage in a dockerfile

Write in your notes: What CVEs (if any) were found? What base image are you using?

    base image: node:24.19.0-alpine
    
    CVEs were: CVE-2026-13149
               CVE-2026-14257 
               CVE-2026-69152
               CVE-2026-69192
               CVE-2026-59873
               and 2-3 more

Task 2: Enable GitHub's Built-in Secret Scanning

GitHub can automatically detect if someone pushes a secret (API key, token, password) to your repo.

1) Go to your repo → Settings → Code security and analysis

2) Enable Secret scanning

3) If available, also enable Push protection — this blocks the push entirely if a secret is detected

That's it — no workflow changes needed. GitHub does this automatically.

Write in your notes:

* What is the difference between secret scanning and push protection?

        |   | Feature       | Secret Scanning                        | Push Protection                         |   |
        |---|---------------|----------------------------------------|-----------------------------------------|---|
        |   | ------------- | -------------------------------------- | --------------------------------------- |   |
        |   | Purpose       | **Find leaked secrets**                | **Prevent secrets from being pushed**   |   |
        |   | When it works | After/during repository scanning       | Before the push reaches GitHub          |   |
        |   | Action        | Creates an alert                       | Blocks the push                         |   |
        |   | Example       | AWS key already exists in repo → alert | You try to push AWS key → push rejected |   |
        |   | Main goal     | Detect & remediate                     | Prevent the leak                        |   |


* What happens if GitHub detects a leaked AWS key in your repo?

- AWS Quarantines the Key: AWS automatically applies a restrictive security policy to the compromised IAM user to block dangerous actions.

- AWS Sends an Alert: You will receive an urgent email and an automated support case is opened in your AWS account.

- Bots Try to Exploit It: Automated hacker bots constantly scrape GitHub and may still harvest the key within seconds, trying to spin up crypto-miners or steal data before the quarantine takes effect.

- Your Next Step: You must immediately deactivate the key in AWS IAM and purge your Git history.


Task 3: Scan Dependencies for Known Vulnerabilities

If your app uses packages (pip, npm, etc.), those packages might have known vulnerabilities.

Add this to your PR pipeline (not the main pipeline):

    - name: Check Dependencies for Vulnerabilities
      uses: actions/dependency-review-action@v4
      with:
        fail-on-severity: critical

This checks any new dependencies added in the PR against a vulnerability database. If a dependency has a critical CVE, the PR check fails.

Test it:

1) Open a PR that adds a package to your app
        
        added lodash package to the package.json

2) Check the Actions tab — did the dependency review run?



Verify: Does the dependency review show up as a check on your PR?

