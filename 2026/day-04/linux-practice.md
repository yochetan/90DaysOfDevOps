Include 2 process commands (ps, top, pgrep, etc.)
ps: report a snapshot of the current processes.
ps
    PID TTY          TIME CMD
   2557 pts/0    00:00:00 bash
   2680 pts/0    00:00:00 ps
ubuntu@ip-172-31-70-56:~$ man ps

top - display Linux processes

pgrep, pkill, pidwait - look up, signal, or wait for processes based on name and other attributes

Include 2 service commands (systemctl status, systemctl list-units, etc.)
systemctl - Control the systemd system and service manager

list-units [PATTERN...]
           List units that systemd currently has in memory. This includes units that are either referenced directly or through a dependency, units that are pinned by applications programmatically, or unitsre active, have pending jobs, or 
have failed are shown; this can be changed with option --all. If one or more PATTERNsre shown are additionally filtered
 by --type= and --state= if those options are specified.ate, values depend on unit type.

 Include 2 log commands (journalctl -u <service>, tail -n 50, etc.)
journalctl - Print log entries from the systemd journal

tail - output the last part of files
-n, --lines=[+]NUM

