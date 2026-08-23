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

Verify: Can you see the vulnerability table in the logs? Did it pass or fail?

Write in your notes: What CVEs (if any) were found? What base image are you using?
