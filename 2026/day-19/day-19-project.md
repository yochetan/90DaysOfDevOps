Task 1: Log Rotation Script

Create log_rotate.sh that:

1) Takes a log directory as an argument (e.g., /var/log/myapp)



2) Compresses .log files older than 7 days using gzip



3) Deletes .gz files older than 30 days



4) Prints how many files were compressed and deleted



5) Exits with an error if the directory doesn't exist
