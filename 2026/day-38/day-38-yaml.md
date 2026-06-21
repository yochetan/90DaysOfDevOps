Task 1: Key-Value Pairs

Create person.yaml that describes yourself with:

name
role
experience_years
learning (a boolean)
Verify: Run cat person.yaml — does it look clean? No tabs?

`person.yml`

    name: Chetan
    role: Devops Engineer
    experience_years: 0
    learning: true

Output:

cat person.yml

    name: Chetan
    
    role: Devops Engineer
    
    experience_years: 0
    
    learning: true

Task 2: Lists
Add to person.yaml:

Write in your notes: What are the two ways to write a list in YAML?

    - block style
    - inline 

tools — a list of 5 DevOps tools you know or are learning

    tools: 
      - GitHub Actions
      - Docker
      - Kubernetes
      - Jenkins
      - Terraform
      
hobbies — a list using the inline format [item1, item2]

    hobbies: ["gym", "gaming", "trekking"]

Task 3: Nested Objects
Create server.yaml that describes a server:

server with nested keys: name, ip, port

    server:
            name: MongoDB server
            ip: 127.0.0.1
            port: 27017
    
database with nested keys: host, name, credentials (nested further: user, password)

     database:
            host: localhost
            name: PortfolioDB
            credentials:
                username: admin
                password: admin123
            
Verify: Try adding a tab instead of spaces — what happens when you validate it?

    - In YAML, tabs are not allowed for indentation. You must use spaces.
    - YAML uses indentation to represent structure, so the specification requires spaces to avoid ambiguity between editors and operating systems.

Task 4: Multi-line Strings
In server.yaml, add a startup_script field using:

The | block style (preserves newlines)

    startup_script: |
      echo "Starting server..."
      sudo systemctl start nginx
      echo "Server started"
  
The > fold style (folds into one line)

    startup_script: >
      echo "Starting server..."
      sudo systemctl start nginx
      echo "Server started"
  
Write in your notes: When would you use | vs >?
    
| (Literal Style)
    
    Preserves line breaks exactly as written.
    Use for:
    - Shell scripts
    - Code snippets
    - Configuration blocks
    - Certificates/keys

> (Folded Style)

    Replaces most line breaks with spaces.
    Use for:
    - Long descriptions
    - Documentation text
    - Multi-line messages or comments

Task 5: Validate Your YAML
1) Install yamllint or use an online validator
2) Validate both your YAML files
3) Intentionally break the indentation — what error do you get?
4) Fix it and validate again

Task 6: Spot the Difference
Read both blocks and write what's wrong with the second one:

    # Block 1 - correct
    name: devops
    tools:
      - docker
      - kubernetes
    # Block 2 - broken
    name: devops
    tools:
    - docker
      - kubernetes

the space is wrong in the second block
