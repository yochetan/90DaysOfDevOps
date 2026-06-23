Task 1: The Problem

Think about a team of 5 developers all pushing code to the same repo manually deploying to production.

Write in your notes:

1) What can go wrong?
        
        - No proper testing and that leaves bug and deployment risks and lot of time wasted.
        - Developers may overwrite each other's changes.
        - Bugs can be introduced into production.
        - Deployment conflicts can occur when multiple developers deploy at the same time.

2) What does "it works on my machine" mean and why is it a real problem?

        - The dependencies are on the developer pc and not on the client/user which leads to bad 
        - Different operating systems
        - Different software versions
        - Missing dependencies
        - Different environment variables or configurations

3) How many times a day can a team safely deploy manually?

        - very few cause doing all manually takes time
        - There is no fixed limit, but manual deployments are slow and error-prone. Most teams can safely perform only a few manual deployments per day because each deployment requires careful checking and human intervention.
        - With CI/CD automation, teams can safely deploy many times per day (sometimes dozens or even hundreds of times) because testing and deployment steps are automated and consistent.

* Conclusion: Manual deployments increase risk and reduce deployment frequency, while automated CI/CD pipelines make deployments faster, safer, and more reliable.
