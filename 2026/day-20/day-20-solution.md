Task 1: Input and Validation

Your script should:

1) Accept the path to a log file as a command-line argument

        log_file="$1"

2) Exit with a clear error message if no argument is provided

        if [ $# -ne 1 ]; then
             echo "Usage: $0 <log_file>"
             exit 1
        fi  

3) Exit with a clear error message if the file doesn't exist

        if [ ! -f "$log_file"; then
               echo "Error: File doesn't exists"
               exit 1
        fi


Task 2: Error Count

1) Count the total number of lines containing the keyword ERROR or Failed

        ERROR_COUNT=$(grep -c "ERROR" "$log_file")

2) Print the total error count to the console

        echo "Total Errors Counts: $ERROR_COUNT"

OUTPUT: 

        ./log_analyzer.sh /var/log/syslog
        Total Error Counts: 4


Task 3: Critical Events

1) Search for lines containing the keyword CRITICAL

        CRTICAL_LINES=$(grep -n "CRITICAL" "log_file")

2) Print those lines along with their line number

       if [ -z "$CRITICAL_LINES" ] ;then
              echo "No critical events found."
       else
              echo "$CRITICAL_LINES"
       fi

OUTPUT:

    ./log_analyzer.sh zookeeper.log 
    Total Error Counts: 13
    --- CRITICAL_EVENTS ---
    No critical events found.


Task 4: Top Error Messages

Extract all lines containing ERROR
Identify the top 5 most common error messages
Display them with their occurrence count, sorted in descending order

    echo "--- Top 5 Error Messages ---"
    
    top_errors=$(grep "ERROR" "$log_file" | awk '{$1=$2=$3=""; print}' | sort | uniq -c | sort -rn | head -5)
    if [ -z "$top_errors" ];then
            echo "No error messages found."
    else
            echo "$top_errors"
    fi


OUTPUT:
    
    ./log_analyzer.sh /var/log/syslog 
    Total Error Counts: 6
    --- CRITICAL_EVENTS ---
    No critical events found.
    --- Top 5 Error Messages ---
          1    2026-04-22 20:27:12.2262 ERROR [CredentialRefresher] Retrieve credentials produced error: no valid credentials could be retrieved for ec2 identity. Default Host Management Err: error calling RequestManagedInstanceRoleToken: AccessDeniedException: Systems Manager's instance management role is not configured for account: 665602132674
          1    2026-04-22 20:27:12.2262 ERROR EC2RoleProvider Failed to connect to Systems Manager with SSM role credentials. error calling RequestManagedInstanceRoleToken: AccessDeniedException: Systems Manager's instance management role is not configured for account: 665602132674
          1    2026-04-22 20:02:00.1892 ERROR [CredentialRefresher] Retrieve credentials produced error: no valid credentials could be retrieved for ec2 identity. Default Host Management Err: error calling RequestManagedInstanceRoleToken: AccessDeniedException: Systems Manager's instance management role is not configured for account: 665602132674
          1    2026-04-22 20:02:00.1892 ERROR EC2RoleProvider Failed to connect to Systems Manager with SSM role credentials. error calling RequestManagedInstanceRoleToken: AccessDeniedException: Systems Manager's instance management role is not configured for account: 665602132674
          1    2026-04-22 19:35:29.0713 ERROR [CredentialRefresher] Retrieve credentials produced error: no valid credentials could be retrieved for ec2 identity. Default Host Management Err: error calling RequestManagedInstanceRoleToken: AccessDeniedException: Systems Manager's instance management role is not configured for account: 665602132674


Task 5: Summary Report

Generate a summary report to a text file named log_report_<date>.txt (e.g., log_report_2026-02-11.txt). The report should include:

Date of analysis
Log file name
Total lines processed
Total error count
Top 5 error messages with their occurrence count
List of critical events with line numbers

    #!/bin/bash
    
    # ==============================
    # Task 1: Input and Validation
    # ==============================
    
    if [ $# -eq 0 ]; then
        echo "Error: No log file provided."
        echo "Usage: $0 <log_file_path>"
        exit 1
    fi
    
    LOG_FILE="$1"
    
    if [ ! -f "$LOG_FILE" ]; then
        echo "Error: File '$LOG_FILE' does not exist."
        exit 1
    fi
    
    if [ ! -r "$LOG_FILE" ]; then
        echo "Error: File '$LOG_FILE' is not readable."
        exit 1
    fi
    
    # ==============================
    # Setup Report File
    # ==============================
    
    DATE=$(date +%F)
    REPORT_FILE="log_report_${DATE}.txt"
    
    # ==============================
    # Analysis Section
    # ==============================
    
    TOTAL_LINES=$(wc -l < "$LOG_FILE")
    
    ERROR_COUNT=$(grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l)
    
    TOP_ERRORS=$(grep -i "ERROR" "$LOG_FILE" \
        | sed 's/.*ERROR[: ]*//' \
        | sort | uniq -c | sort -nr | head -5)
    
    CRITICAL_EVENTS=$(grep -n "CRITICAL" "$LOG_FILE")
    
    # ==============================
    # Generate Report
    # ==============================
    
    {
    echo "======================================"
    echo "        LOG ANALYSIS REPORT"
    echo "======================================"
    echo "Date of Analysis : $DATE"
    echo "Log File         : $LOG_FILE"
    echo ""
    
    echo "------ SUMMARY ------"
    echo "Total Lines Processed : $TOTAL_LINES"
    echo "Total Error Count     : $ERROR_COUNT"
    echo ""
    
    echo "------ TOP 5 ERROR MESSAGES ------"
    if [ -z "$TOP_ERRORS" ]; then
        echo "No error messages found."
    else
        echo "$TOP_ERRORS"
    fi
    echo ""
    
    echo "------ CRITICAL EVENTS (with line numbers) ------"
    if [ -z "$CRITICAL_EVENTS" ]; then
        echo "No critical events found."
    else
        echo "$CRITICAL_EVENTS"
    fi
    echo ""
    
    echo "======================================"
    echo "Report generated successfully."
    echo "======================================"
    
    } > "$REPORT_FILE"
    
    # ==============================
    # Done
    # ==============================
    
    echo "Report saved to: $REPORT_FILE"

OUTPUT:

    ./log_analyzer1.sh zookeeper.log 
    Report saved to: log_report_2026-04-22.txt

    cat log_report_2026-04-22.txt 
    ======================================
            LOG ANALYSIS REPORT
    ======================================
    Date of Analysis : 2026-04-22
    Log File         : zookeeper.log
    
    ------ SUMMARY ------
    Total Lines Processed : 2000
    Total Error Count     : 305
    
    ------ TOP 5 ERROR MESSAGES ------
          1 [LearnerHandler-/10.10.34.13:57617:LearnerHandler@562] - Unexpected exception causing shutdown while sock still open
          1 [LearnerHandler-/10.10.34.13:57438:LearnerHandler@562] - Unexpected exception causing shutdown while sock still open
          1 [LearnerHandler-/10.10.34.13:57354:LearnerHandler@562] - Unexpected exception causing shutdown while sock still open
          1 [LearnerHandler-/10.10.34.12:59480:LearnerHandler@562] - Unexpected exception causing shutdown while sock still open
          1 [LearnerHandler-/10.10.34.12:59455:LearnerHandler@562] - Unexpected exception causing shutdown while sock still open
    
    ------ CRITICAL EVENTS (with line numbers) ------
    No critical events found.
    
    ======================================
    Report generated successfully.
