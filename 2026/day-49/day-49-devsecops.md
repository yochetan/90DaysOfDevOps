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
