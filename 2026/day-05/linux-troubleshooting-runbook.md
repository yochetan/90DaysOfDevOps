Run and record output for at least 8 commands (save snippets in your runbook)
Environment basics (2): uname -a, lsb_release -a (or cat /etc/os-release)

uname -a
Linux ip-172-31-70-56 6.17.0-1009-aws #9~24.04.2-Ubuntu SMP Fri Mar  6 23:50:29 UTC 2026 x86_64 x86_64 x86_64 GNU/Linux
uname - print system information
-a, --all
              print all information, in the following order, except omit -p and -i if unknown:

lsb_release - print distribution-specific information (minimal implementation).

cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.3 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo


Filesystem sanity (2): create a throwaway folder and file, e.g., mkdir /tmp/runbook-demo, cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo

cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo
total 4
-rw-r--r-- 1 ubuntu ubuntu 221 Mar 30 19:05 hosts-copy [didn't got what happened]


CPU / Memory (2): top/htop/ps -o pid,pcpu,pmem,comm -p <pid>, free -h, vm_stat (mac)

 top - display Linux processes
 htop, pcp-htop - interactive process viewer
 ps - report a snapshot of the current processes.
  ps -o pid 
    PID
   2557
   3770

  ps -o pcpu
  %CPU
   0.0
   0.0

  ps -o pmem
  %MEM
   0.5
   0.4

  ps -o comm
  COMMAND
    bash
    ps
    
free - Display amount of free and used memory in the system
-h, --human
              Show all output fields automatically scaled to shortest three digit unit and display the units of print out.  Following units are used.

                B = bytes
                Ki = kibibyte
                Mi = mebibyte
                Gi = gibibyte
                Ti = tebibyte
                Pi = pebibyte
  free -h
               total        used        free      shared  buff/cache   available
Mem:           911Mi       406Mi       228Mi       2.7Mi       435Mi       505Mi
Swap:             0B          0B          0B


Disk / IO (2): df -h, du -sh /var/log, iostat/vmstat/dstat
df - report file system space usage
-h, --human-readable
              print sizes in powers of 1024 (e.g., 1023M)

du - estimate file space usage
du -sh
152K 

iostat - Report Central Processing Unit (CPU) statistics and input/output statistics for devices and partitions.
vmstat - Report virtual memory statistics
pcp-dstat - versatile tool for generating system resource statistic


Network (2): ss -tulpn/netstat -tulpn, curl -I <service-endpoint>/ping

ss - another utility to investigate sockets

netstat - Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships

curl - transfer a URL

ping - send ICMP ECHO_REQUEST to network hosts


Logs (2): journalctl -u <service> -n 50, tail -n 50 /var/log/<file>.log

journalctl - Print log entries from the systemd journal
-u, --unit=UNIT|PATTERN
           Show messages for the specified systemd unit UNIT (such as a service unit), or for any of the units matched by PATTERN. If a pattern is specified, a list of unit names found in the journal is
           compared with the specified pattern and all that match are used. For each unit name, a match is added for messages from the unit ("_SYSTEMD_UNIT=UNIT"), along with additional matches for
           messages from systemd and messages about coredumps for the specified unit. A match is also added for "_SYSTEMD_SLICE=UNIT", such that if the provided UNIT is a systemd.slice(5) unit, all logs of
           children of the slice will be shown.

 tail - output the last part of files

 



