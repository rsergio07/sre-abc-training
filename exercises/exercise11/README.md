
# Table of Contents

- [Observability Approach](#observability-approach)
  - [What Are the Golden Signals?](#what-are-the-golden-signals)
  - [Latency](#latency)
  - [Traffic](#traffic)
  - [Errors](#errors)
  - [Saturation](#saturation)
- [Deployment](#deployment)
- [Tip for Infrastructure as Code (IaC) with Ansible](#tip-for-infrastructure-as-code-iac-with-ansible)
- [Final Objective](#final-objective)

# Observability approach

The **Golden Signals** approach is a fundamental monitoring framework in **Site Reliability Engineering (SRE)**. These signals provide key insights into system health, helping engineers quickly identify, diagnose, and resolve issues in production. Google's SRE teams developed the Golden Signals framework to define a standard set of metrics essential for monitoring system reliability.

![Logs](<../exercise10/Infra.png>)
From infrastucture point of view all the elements are going to be the same because the new dashboards are going to be inside Grafana.

## What Are the Golden Signals?

The Golden Signals consist of four primary metrics:

1. **Latency**: The time it takes to service a request. Latency includes both successful and failed requests, with a focus on measuring the time for successful ones as a primary indicator.

  > [!TIP]
  > Create a Granafa panel with `sum(rate(otel_collector_span_metrics_duration_milliseconds_bucket[5m])) by (span_name)` using the `otel_collector_span_metrics_duration_milliseconds_bucket` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result1.png" alt="Prometheus result" height="150" />


2. **Traffic**: The amount of demand on the system, such as the number of requests per second. Traffic helps SREs understand usage patterns and anticipate scalability needs.

  > [!TIP]
  > Create a Granafa panel with `sum(rate(container_network_receive_bytes_total[5m])) by (container_label_k8s_app) from cadvisor` using the `container_network_receive_bytes_total` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result2.png" alt="Prometheus result" height="150" />

  > [!TIP]
  > Create a Granafa panel with `sum(rate(container_network_transmit_bytes_total[5m])) by (span_name)` using the `container_network_transmit_bytes_total` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result3.png" alt="Prometheus result" height="150" />

  > [!TIP]
  > Create a Granafa panel with `sum(rate(container_network_receive_errors_total[5m])) by (span_name)` using the `container_network_receive_errors_total` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result4.png" alt="Prometheus result" height="150" />

  > [!TIP]
  > Create a Granafa panel with `sum(rate(container_network_transmit_errors_total[5m])) by (span_name)` using the `container_network_transmit_errors_total` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result5.png" alt="Prometheus result" height="150" />

3. **Errors**: The rate of failed requests. Monitoring error rates allows engineers to see if users are encountering issues, which can be critical for user experience.

  > [!TIP]
  > Create a Granafa panel with `sum(rate(container_scrape_error[5m])) by (job)` using the `container_scrape_error` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result6.png" alt="Prometheus result" height="150" />

  > [!TIP]
  > Create a Granafa panel with `{service_name=“unknown_service”} |= 'err'` using the logs line with the `err` level 
  > <img src="images/grafana_result1.png" alt="Grafana result" height="150" />

4. **Saturation**: The overall capacity and resource usage of the system, such as CPU or memory usage. Saturation helps identify potential bottlenecks before they cause outages.

  > [!TIP]
  > Create a Granafa panel with `sum(rate(container_cpu_system_seconds_total[5m])) by (job)` using the `container_cpu_system_seconds_total` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result7.png" alt="Prometheus result" height="150" />

  > [!TIP]
  > Create a Granafa panel with `sum(rate(container_label_io_kubernetes_container_name[5m])) by (job)` using the `container_label_io_kubernetes_container_name` metric from the Cadvisor service.
  > The output shoud be like this:
  > <img src="images/prometheus_result8.png" alt="Prometheus result" height="150" />

# Deployment
Before deploy all the new staff it's important to clean the changes from the previous exercises and then apply the new settings wih short program like this one:
```bash
#!/bin/bash

kubectl delete ns application
kubectl delete ns opentelemetry
kubectl delete ns monitoring
kubectl delete pv --all 
kubectl delete pvc --all 
sleep 5;

echo "-------------------------------------------------------------------------"
echo "Start creating"
echo "-------------------------------------------------------------------------"
kubectl apply -f ../exercise10/storage.yaml;
kubectl apply -f ../exercise10/deployment.yaml;
kubectl apply -f ../exercise10/otel-collector.yaml;
kubectl apply -f ../exercise8/jaeger.yaml;
kubectl apply -f ../exercise9/prometheus.yaml;
kubectl apply -f ../exercise10/grafana-loki.yaml;
kubectl apply -f ./grafana.yaml;
echo "-------------------------------------------------------------------------"
echo "wait"
echo "-------------------------------------------------------------------------"
sleep 5;
kubectl get pods -A
```
# Tip for Infrastructure as Code (IaC) with Ansible

> [!TIP]
> A more efficient **Infrastructure as Code (IaC)** approach can be implemented with Ansible to apply the new configuration and start its service in Minikube. An [example](./infra.yaml) of how to structure a YAML playbook to achieve this.

> 2. **Run the Playbook**
> ```bash
> ansible-playbook -i ../exercise4.1/ansible_quickstart/inventory.ini infra.yaml
> minikube service grafana-service -n monitoring
> ```

---
# Final Objective
At the end of this document, you should accomplished this:
> [!IMPORTANT]
> These signals offer a holistic view of system performance, enabling SREs to quickly address critical reliability issues. Here is a preview of the dashboard 
> 
> <img src="images/Dashboard.png" alt="Dashboard" height="150" />
> <img src="images/Dashboard1.png" alt="Dashboard" height="150" />
> 
> At [New Dashboard](grafana.yaml) is an example of all Grafana setting which include the new dashboard configuration