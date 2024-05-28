**PROJECT 5:**

**Project Description:**

Monitoring using Prometheus & Grafana


**Goals:**

Creating a Dashboard to monitor the Kubernetes cluster and pipelines


**Technologies Used:**

1\. Prometheus

2\. Grafana


**Steps:**

1\. Introduction to Prometheus and Grafana

2\. Prometheus and Grafana Setup

3\. Monitoring using Prometheus Dashboard Visualization using Grafana

4\. Creating a Dashboard to monitor the Pipeline




**Setting up monitoring for your Kubernetes cluster and pipelines using Prometheus and Grafana on AWS**

**Understanding Prometheus and Grafana:**

- **Prometheus:** An open-source monitoring system that scrapes metrics from various targets (like Kubernetes components, applications, and infrastructure). It stores these metrics in a time-series database for analysis and visualization.
- **Grafana:** An open-source platform for creating informative dashboards and visualizations of your monitoring data collected by Prometheus. It allows you to create interactive dashboards that display key metrics and identify potential issues.


**Setting Up Prometheus and Grafana on AWS:**

1. **Launch an Amazon EC2 Instance:**
    - Choose an appropriate Amazon Machine Image (AMI) for your needs. A popular option is Ubuntu Server 20.04 LTS.
    - Select an instance type with enough resources to handle Prometheus and Grafana workload.
    - Configure security groups to allow inbound traffic on ports 9090 (Prometheus) and 3000 (Grafana).
2. **Install Prometheus:**
    - Connect to your EC2 instance using SSH.
    - Update package lists:  `sudo apt update`
    - Install required packages: `sudo apt install prometheus node_exporter` (node_exporter collects system metrics)
    - Download the latest Prometheus binary from the official website: <https://prometheus.io/download/>
    - Unpack the downloaded archive and move the binary to a suitable location `/usr/local/bin/prometheus`.
    - Create a systemd service file for Prometheus `/etc/systemd/system/prometheus.service`:

~~~
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/data/prometheus

[Install]
WantedBy=multi-user.target
~~~

   - Create a directory for Prometheus data storage `/data/prometheus`.
   - Set ownership and permissions for the data directory: `sudo chown -R prometheus:prometheus /data/prometheus`
   - Create a configuration file for Prometheus `/etc/prometheus/prometheus.yml`:

YAML
~~~
global:
scrape_interval: 15s # Scrape targets every 15 seconds

scrape_configs:
- job_name: kubernetes
scheme: https # Use HTTPS if your Kubernetes API server has TLS enabled

# Replace with your actual Kubernetes API server address and port
server: https://<your-kubernetes-api-server>:6443

# Use a Kubernetes service account to scrape metrics from the API server
bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

# Define additional scrape targets for other components as needed
- job_name: node_exporter

static_configs:
- targets: ['localhost:9100'] # Default node_exporter port
~~~

   - Replace &lt;your-kubernetes-api-server&gt; with your actual Kubernetes API server address. You'll need a Kubernetes service account with permissions to scrape API server metrics.
   - Reload systemd and start Prometheus: `sudo systemctl daemon-reload && sudo systemctl start prometheus`
   - Verify Prometheus is running: `sudo systemctl status Prometheus`

1. **Install Grafana:**
    - Install required packages: sudo apt install grafana
    - Create a systemd service file for Grafana `/etc/systemd/system/grafana.service`:
~~~
[Unit]
Description=Grafana
After=network.target

[Service]
User=grafana
Group=grafana
Type=simple
ExecStart=/usr/sbin/grafana-server

[Install]
WantedBy=multi-user.target
~~~

   - Reload systemd and start Grafana: `sudo systemctl daemon-reload && sudo systemctl start grafana`

   - Verify Grafana is running: `sudo systemctl status grafana`

**Monitoring Using Prometheus Dashboard:**
1. **Access Prometheus:** Open `http://<your-ec2-instance-public-ip>:9090/` in your web browser.
2. **Explore Targets:** Go to the **Targets** menu to see if Prometheus is scraping metrics from your Kubernetes cluster and node_exporter.

**Grafana Visualization**
Now that Prometheus is up and running, collecting metrics from your Kubernetes cluster and node_exporter, it's time to leverage Grafana to visualize these metrics and create informative dashboards.

1. **Access Grafana:** Open `http://<your-ec2-instance-public-ip>:3000/` in your web browser.
2. **Create a Prometheus Data Source:**
    - In Grafana, go to **Configuration** -> **Data Sources**.
    - Click **Add data source**.
    - Select **Prometheus** as the type.
    - Give your data source a name ("Prometheus").
    - In the **URL** field, enter the URL of your Prometheus server (<http://localhost:9090/> if running on the same machine).
    - Click **Save & test**.
3. **Explore Prometheus Metrics:**
    - In the Grafana sidebar, under **Explore**, click on your Prometheus data source.
    - The explore interface allows you to query Prometheus for specific metrics and visualize them in various graphs.
    - Experiment with the query bar to explore available metrics (e.g., kube_pod_info{namespace="default"}).
    - Use the panel editor on the right to customize the graph type (line, graph, heatmap, etc.) and other visualization options.
4. **Creating a Dashboard to Monitor the Kubernetes Cluster:**
    - In the Grafana sidebar, click **Dashboards** -> **New dashboard**.
    - Choose a layout for your dashboard (e.g., row-based).
    - Start adding panels by clicking the **\+ Add Panel** button.
    - Select **Prometheus** as the data source for each panel.
    - Use the query bar in each panel to define the specific metrics you want to visualize. Here are some examples:
        - **CPU Usage:** `node_cpu_usage` (from node_exporter)
        - **Memory Usage:** `node_memory_MemTotal and node_memory_MemFre`e (from node_exporter)
        - **ReplicaSet Status:** `kube_replicaset_status_replicas{replicaset!=""}` (from Kubernetes API server)
        - **Pod Status:** `kube_pod_status_phase{pod!=""}` (from Kubernetes API server)
        - **Container Resources:** `container_cpu_usage_seconds_total` and `container_memory_usage_bytes` (from Kubernetes API server)
    - Customize each panel with appropriate display options (graph type, labels, thresholds) to create a clear and informative view.

5. **Creating a Dashboard to Monitor the Pipeline:**

**Pipeline Metrics Exported by the Pipeline Tool:** - Considering pipeline tool as Jenkins, configure Prometheus to scrape the metrics and visualize them in Grafana.
Once you've identified relevant metrics, you can configure Prometheus to scrape them from the pipeline tool's API endpoint. Here's a general structure for a Prometheus scrape configuration targeting a pipeline tool API:
With the Jenkins Prometheus Plugin installed, you can configure Prometheus to scrape these metrics. Add the following code to the `prometheus.yml` file

YAML
~~~
- job_name: "jenkins"
scheme: https # Use HTTPS if your Jenkins has TLS enabled

# Replace with the actual Jenkins server URL and port
server: https://<your-jenkins-server-url>:<port>

# Path to the Prometheus endpoint exposed by the plugin (default: /prometheus)
path: /prometheus

# Define specific metric scraping rules (adjust based on your needs)
scrape_targets:
- targets: ['/jobs?status=SUCCESS'] # Example: scrape success rate metric (adjust)
- targets: ['/jobs?status=FAILURE'] # Example: scrape failure rate metric (adjust)
- targets: ['/queue'] # Example: scrape queue length metric

# Add additional targets for other metrics based on your needs
~~~

**Visualizing Pipeline Metrics in Grafana:**

Once Prometheus is scraping pipeline metrics, you can create dashboards in Grafana to visualize them. Here's how:

- **Access Grafana:** Open `http://<your-ec2-instance-public-ip>:3000/` in your web browser.
- **Create a Dashboard:** Go to **Dashboards** -> **New dashboard**.
- **Add Panels:** Click the **\+ Add Panel** button and select **Prometheus** as the data source.
- **Define Metrics:** In the query bar of each panel, define the specific pipeline metrics you want to visualize. Here are some examples:
  - **Job Success Rate:** `rate(kube_job_success{pipeline_name="your-pipeline"}[5m])` (assuming your pipeline tool exposes a "kube_job_success" metric)
  - **Job Execution Time (Average):** `avg(kube_job_duration_seconds{pipeline_name="your-pipeline"}[5m])`
  - **Queued Jobs:** `kube_job_queued{pipeline_name="your-pipeline"}`
- **Customize Panels:** Adjust the graph type, labels, thresholds, and other display options to create an informative view.
