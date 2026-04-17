Task 1: Log Rotation Script

Create log_rotate.sh that:

1) Takes a log directory as an argument (e.g., /var/log/myapp)

        # Check if argument is provided
        if [ $# -ne 1 ]; then
            echo "Usage: $0 <log_directory>"
            exit 1
        fi

        LOG_DIR="$1"

        # Check if directory exists
        if [ ! -d "$LOG_DIR" ]; then
            echo "Error: Directory does not exist!"
            exit 1
        fi

2) Compresses .log files older than 7 days using gzip

        find "$LOG_DIR" -type f -name "*.log" -mtime +7 -exec gzip {} \;

3) Deletes .gz files older than 30 days

        find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -exec rm -f {} \;

4) Prints how many files were compressed and deleted
        
       # Count files before processing
       compressed_count=$(find "$LOG_DIR" -type f -name "*.log" -mtime +7 | wc -l)
       deleted_count=$(find "$LOG_DIR" -type f -name "*.gz" -mtime +30 | wc -l)

       echo "Log rotation completed."
       echo "Files compressed: $compressed_count"
       echo "Files deleted: $deleted_count"

6) Exits with an error if the directory doesn't exist

        exit 0

OUTPUT:

        ./log_rotate.sh /var/log/myapp/
        Log rotation completed.
        Files compressed: 2
        Files deleted: 0


Task 2: Server Backup Script

Create backup.sh that:

1) Takes a source directory and backup destination as arguments

        usage() {
            echo "Usage: $0 <source_directory> <backup_destination>"
            exit 1
        }
        
        # Check arguments
        if [ $# -ne 2 ]; then
            usage
        fi
        
        SOURCE_DIR="$1"
        DEST_DIR="$2"

2) Creates a timestamped .tar.gz archive (e.g., backup-2026-02-08.tar.gz)

        TIMESTAMP=$(date +"%Y-%m-%d")
        ARCHIVE_NAME="backup-$TIMESTAMP.tar.gz"
        ARCHIVE_PATH="$DEST_DIR/$ARCHIVE_NAME"

3) Verifies the archive was created successfully

        # Create archive
        echo "Creating backup..."
        tar -czf "$ARCHIVE_PATH" "$SOURCE_DIR" 2>/dev/null
        
        # Verify archive creation
        if [ ! -f "$ARCHIVE_PATH" ]; then
            echo "Error: Backup failed!"
            exit 1
        fi

4) Prints archive name and size

        # Get archive size
        ARCHIVE_SIZE=$(du -h "$ARCHIVE_PATH" | cut -f1)
        
        echo "Backup successful!"
        echo "Archive: $ARCHIVE_NAME"
        echo "Size: $ARCHIVE_SIZE"

5) Deletes backups older than 14 days from the destination

        # Delete backups older than 14 days
        echo "Cleaning old backups..."
        DELETED_COUNT=$(find "$DEST_DIR" -name "backup-*.tar.gz" -mtime +14 -type f | wc -l)
        
        find "$DEST_DIR" -name "backup-*.tar.gz" -mtime +14 -type f -exec rm -f {} \;
        
        echo "Deleted $DELETED_COUNT old backup(s)."

6) Handles errors — exit if source doesn't exist

        echo 0

OUTPUT:

        ./backupp.sh /home/ubuntu/data/ /home/ubuntu/backups/
        Creating backup...
        Backup successful!
        Archive: backup-2026-04-17.tar.gz
        Size: 4.0K
        Cleaning old backups...
        Deleted 0 old backup(s).


Task 3: Crontab

1) Read: crontab -l — what's currently scheduled?

        no crontab for ubuntu

2) Understand cron syntax:

        * * * * *  command
        │ │ │ │ │
        │ │ │ │ └── Day of week (0-7)
        │ │ │ └──── Month (1-12)
        │ │ └────── Day of month (1-31)
        │ └──────── Hour (0-23)
        └────────── Minute (0-59)

3) Write cron entries (in your markdown, don't apply if unsure) for:

Run log_rotate.sh every day at 2 AM

      0 2 * * * /path/to/log_rotate.sh /var/log/myapp

Run backup.sh every Sunday at 3 AM

        0 3 * * 0 /path/to/backup.sh /home/ubuntu/data /home/ubuntu/backups

Run a health check script every 5 minutes

        0/5 * * * * /path/to/health_check.sh


Task 4: Combine — Scheduled Maintenance Script

Create maintenance.sh that:

1) Calls your log rotation function

        # Import the function file
        source ./log_rotate.sh
        # Call the imported function and log output
        log "Starting log rotation for $LOG_DIR..."
        log_rotate "$LOG_DIR" >> "$LOG_FILE" 2>&1

3) Calls your backup function

        # Import the function file
        source ./backup.sh
        # Call the imported function and log output
        log "Starting backup from $BACKUP_SOURCE to $BACKUP_DEST..."
        backup "$BACKUP_SOURCE" "BACKUP_DEST" >> "$LOG_FILE" 2>&1

4) Logs all output to /var/log/maintenance.log with timestamps

        log() {
            echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
        }

5) Write the cron entry to run it daily at 1 AM

        0 1 * * * /path/to/maintainance.sh <log_directory> <backup_source> <backup_destination>

CODE:

        #!/bin/bash
        
        LOG_FILE="/var/log/maintenance.log"
        
        LOG_DIR="$1"
        BACKUP_SOURCE="$2"
        BACKUP_DEST="$3"
        
        # Import functions
        source ./log_rotate.sh
        source ./backupp.sh
        
        # Logging function
        log() {
            echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
        }
        
        # ==============================
        # Validation
        # ==============================
        if [ $# -ne 3 ]; then
            log "ERROR: Usage: $0 <log_directory> <backup_source> <backup_destination>"
            exit 1
        fi
        
        if [ ! -d "$LOG_DIR" ]; then
            log "ERROR: Log directory does not exist: $LOG_DIR"
            exit 1
        fi
        
        if [ ! -d "$BACKUP_SOURCE" ]; then
            log "ERROR: Backup source does not exist: $BACKUP_SOURCE"
            exit 1
        fi
        
        if [ ! -d "$BACKUP_DEST" ]; then
            log "ERROR: Backup destination does not exist: $BACKUP_DEST"
            exit 1
        fi
        
        log "===== Maintenance Started ====="
        
        # ==============================
        # Log Rotation
        # ==============================
        log "Starting log rotation for $LOG_DIR..."
        log_rotate "$LOG_DIR" >> "$LOG_FILE" 2>&1
        
        # ==============================
        # Backup
        # ==============================
        log "Starting backup from $BACKUP_SOURCE to $BACKUP_DEST..."
        backup "$BACKUP_SOURCE" "$BACKUP_DEST" >> "$LOG_FILE" 2>&1
        
        # ==============================
        # Done
        # ==============================
        log "===== Maintenance Completed ====="
        
        exit 0

OUTPUT: 

        ./maintainance.sh /var/log/myapp/ /home/ubuntu/data/ /home/ubuntu/backups/ 
        
        we get this when we cat ./maintainance.sh
        [2026-04-17 21:13:06] ===== Maintenance Started =====
        [2026-04-17 21:13:06] Starting log rotation for /var/log/myapp/...
        Log rotation completed.
        Files compressed: 0
        Files deleted: 0
        [2026-04-17 21:13:06] Starting backup from /home/ubuntu/data/ to /home/ubuntu/backups/...
        Creating backup...
        Backup successful!
        Archive: backup-2026-04-17.tar.gz
        Size: 4.0K
        Cleaning old backups...
        Deleted 0 old backup(s).
        [2026-04-17 21:13:06] ===== Maintenance Completed =====

