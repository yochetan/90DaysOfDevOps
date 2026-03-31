Question: How do you check if the 'nginx' service is running?

first how to install it
sudo apt update -> this will update the required packages for any installation
sudo apt install nginx -> this will install the service

step 1: Check service status
systemctl status nginx
-> this will let us know if the service is stopped or working and some things below is the output and the service currently on
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-03-31 20:05:10 UTC; 2min 17s ago
       Docs: man:nginx(8)
    Process: 2372 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2374 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 2405 (nginx)
      Tasks: 3 (limit: 1004)
     Memory: 2.4M (peak: 5.3M)
        CPU: 25ms
     CGroup: /system.slice/nginx.service
             ├─2405 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─2407 "nginx: worker process"
             └─2408 "nginx: worker process"

Mar 31 20:05:10 ip-172-31-29-59 systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Mar 31 20:05:10 ip-172-31-29-59 systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.


Step 2: If service is not found, list all services
systemctl list-units --type=service
-> this command will list all the services which are active but (running/exited) something like that


Step 3: Check if service is enabled on boot
systemctl is-enabled nginx
-> this will tell is it enabled or disabled 

Scenario:
A web application service called 'myapp' failed to start after a server reboot.
What commands would you run to diagnose the issue?
Write at least 4 commands in order.

1) systemctl status myapp
2) journalctl -u myapp -n 50 [this will be give the logs of the myapp] / but for example and real base this is how it will look like
    Mar 31 20:05:10 ip-172-31-29-59 systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
    Mar 31 20:05:10 ip-172-31-29-59 systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
    Mar 31 20:15:45 ip-172-31-29-59 systemd[1]: Stopping nginx.service - A high performance web server and a reverse proxy server...
    Mar 31 20:15:45 ip-172-31-29-59 systemd[1]: nginx.service: Deactivated successfully.
    Mar 31 20:15:45 ip-172-31-29-59 systemd[1]: Stopped nginx.service - A high performance web server and a reverse proxy server.
3) systemctl is-enabled myapp
4) sudo systemctl start nginx [if both are dead]

