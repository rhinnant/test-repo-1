# Instruction on how to setup my monitoring app tool


DevOps Monitor
User Guide
Live Kubernetes metrics, pod status, and log viewer — built with Django

1. What Is DevOps Monitor?
DevOps Monitor is a web-based dashboard built into your Django application that gives you a live view of your Kubernetes cluster. It combines the metrics power of Prometheus with the log aggregation of Loki — all in a single dark-themed UI, without needing to open Grafana or run kubectl commands.

Feature	Description
📊 Dashboard	Live CPU, memory, pod count metrics with auto-updating line charts
🟢 Pod Status	Table of all pods across all namespaces with ready/total counts
📋 Live Logs	Real-time log viewer pulling from Loki, filterable by namespace
🔴 Color Coding	Green = healthy, Yellow = warning (>60%), Red = critical (>80%)
⏱ Auto Refresh	Metrics every 15s, logs every 20s, pods every 30s automatically

Requirement: first start minikube 
2. How to Start the Monitor
The monitor requires three services to be running before you launch Django. Open three separate terminal windows:
Terminal 1 — Expose Prometheus
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
💡 Keep this terminal open. Prometheus provides all CPU and memory metrics.
Terminal 2 — Expose Loki
kubectl port-forward -n monitoring svc/loki 3100:3100
💡 Keep this terminal open. Loki provides the live log data.
Terminal 3 — Start Django
cd ~/Backup/Devnetproject-main
source venv/bin/activate
python3 manage.py runserver

✅ All three terminals must stay open while you use the monitor. Closing any one of them will cause that section of the dashboard to show N/A or no data.
Open the Dashboard
Once all three services are running, open your browser and go to:
http://localhost:8000/monitor/
You should see the dark-themed dashboard load with live data within a few seconds.

3. Dashboard Walkthrough
3.1 Sidebar Navigation
The left sidebar has three navigation items:
•	Dashboard — the main overview with metrics cards and charts (default view)
•	Pods — scrolls you to the pod status table
•	Logs — scrolls you to the live log viewer

3.2 Top Bar
The top bar shows the current section title and a pulsing green dot indicating the dashboard is live and auto-refreshing. Metrics update automatically — you do not need to refresh the page.
3.3 Metric Cards
Four cards across the top of the dashboard show your key cluster health indicators at a glance:

Metric	What It Means	Color Threshold
CPU Usage %	Percentage of CPU being used across all cluster nodes	Green <60%, Yellow 60-80%, Red >80%
Memory Usage %	Percentage of RAM being used across all cluster nodes	Green <60%, Yellow 60-80%, Red >80%
Pods Running	Total count of pods in Running phase across all namespaces	Green always
Pods Pending	Total count of pods waiting to be scheduled	Yellow always — investigate if >0

The card values change color automatically based on severity so you can spot problems instantly without reading numbers.
3.4 Live Charts
Below the metric cards are two line charts that build up over time as you keep the dashboard open:
•	CPU Usage % chart — blue line, shows the last 10 readings
•	Memory Usage % chart — green line, shows the last 10 readings
Each new data point is added every 15 seconds. The charts are most useful after leaving the dashboard open for a few minutes so you can see trends.
3.5 Pod Status Table
The pod table shows every pod in your cluster across all namespaces. Columns:
•	Pod Name — the full Kubernetes pod name
•	Namespace — color-coded in blue (monitoring, argocd, dev, kube-system)
•	Status — colored badge: green for Running, yellow for Pending, red for Failed
•	Ready — shows ready containers vs total (e.g. 3/3 means fully healthy)

✅ If a pod shows 2/3 Ready, one of its containers is not healthy. Note the pod name and run: kubectl describe pod <name> -n <namespace> to investigate.
3.6 Live Log Viewer
The log viewer at the bottom pulls real-time logs from Loki. You can switch between namespaces using the dropdown:

Namespace	What You'll See
monitoring	Prometheus, Grafana, Alertmanager, Loki logs
argocd	ArgoCD sync, deploy, and controller logs
dev	Django application logs
kube-system	CoreDNS, kubelet, API server logs

Logs are color coded by severity:
•	White/grey — normal info logs
•	Yellow — warning logs (contains 'warn')
•	Red — error logs (contains 'error' or 'err=')

The viewer shows the 50 most recent log lines, newest first. It refreshes automatically every 20 seconds.

4. How to Read the Data
4.1 Is My Cluster Healthy?
A healthy cluster should show:
•	CPU and Memory both green (under 60%)
•	Pods Running count matches your expected number of pods
•	Pods Pending showing 0
•	All pods in the table showing Running status and full Ready count (e.g. 2/2, 3/3, 1/1)

✅ Your cluster has 14 CPUs and ~7.8GB RAM available. With the full monitoring stack running, expect CPU around 5-15% and memory around 40-60% under normal conditions.
4.2 Something Looks Wrong — What to Do
If you see red or yellow values:
1.	Check which pods are not Running in the pod table
2.	Note the pod name and namespace
3.	Switch the log viewer to that namespace to see recent errors
4.	Run kubectl describe pod <name> -n <namespace> for full details
5.	Run kubectl logs <name> -n <namespace> --tail=50 for container logs

4.3 Understanding etcd Alerts
You may see error log lines mentioning etcdMembersDown or etcdInsufficientMembers. This is expected on a single-node Minikube cluster and does not indicate a real problem. etcd requires multiple members for quorum, which is only available in production multi-node clusters.
4.4 N/A Values
If a metric card shows N/A instead of a number, it means the dashboard could not reach Prometheus. This happens when:
•	The Prometheus port-forward is not running (Terminal 1 was closed)
•	Prometheus itself is not healthy
Fix: Restart the port-forward in Terminal 1 and wait up to 15 seconds for the dashboard to pick it up automatically.

5. Auto-Refresh Schedule
The dashboard refreshes data automatically. You never need to reload the page manually:

Data	Refresh Interval	Source
Metric cards + charts	Every 15 seconds	Prometheus API
Live log viewer	Every 20 seconds	Loki API
Pod status table	Every 30 seconds	kubectl / Kubernetes API


6. Troubleshooting
Dashboard shows N/A for all metrics
Cause: Prometheus port-forward is not running.
Fix:
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
Log viewer shows 'No logs found'
Cause: Loki port-forward is not running, or the selected namespace has no recent logs.
Fix:
kubectl port-forward -n monitoring svc/loki 3100:3100
Then switch to a different namespace in the dropdown and wait 20 seconds for the auto-refresh.
Pod table shows 'No pods found'
Cause: kubectl is not configured or Minikube is not running.
Fix:
minikube status
kubectl get pods --all-namespaces
Django won't start — ModuleNotFoundError
Cause: Running Django outside the virtual environment.
Fix:
source venv/bin/activate
pip install -r requirements.txt
python3 manage.py runserver
Port already in use
sudo lsof -i :9090    # find what's using Prometheus port
sudo lsof -i :3100    # find what's using Loki port
sudo lsof -i :8000    # find what's using Django port

7. Quick Reference


 
 # start minikube 
 minikube start
 
Start Everything
# Terminal 1
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090

# Terminal 2
kubectl port-forward -n monitoring svc/loki 3100:3100

# Terminal 3
cd ~/Backup/Devnetproject-main && source venv/bin/activate && python3 manage.py runserver

Open the Dashboard
http://localhost:8000/monitor/

API Endpoints
The monitor exposes three internal API endpoints used by the dashboard:
/monitor/api/metrics/    — CPU, memory, pod counts from Prometheus
/monitor/api/pods/       — Pod list from Kubernetes API
/monitor/api/logs/       — Log stream from Loki
💡 You can open these URLs directly in your browser to see the raw JSON data.
