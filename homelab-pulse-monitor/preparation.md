# Preparation: Homelab Pulse Monitor

As of November 1, 2026. Planned; no application code is available yet.

## Goal

An internal dashboard aggregates the status of actual Homelab services. It demonstrates API integration and observability. Grafana remains the tool of choice for detailed metrics and time series.

## Selection

| Area | Recommendation | Why? |
| --- | --- | --- |
| Language | C# / .NET 10 | Same language; focus is on interfaces and reliability. |
| UI | Blazor Web App with Interactive Server | Internal browser-based dashboard with regularly updated statuses. |
| Backend | ASP.NET Core, HTTP clients, and BackgroundService if needed | Centrally query APIs, set timeouts, and temporarily cache results. |
| Data Storage | No dedicated database initially | Current status and existing time series are sufficient. |
| Metrics | Prometheus, once set up | Reuse existing data. |
| Additional Sources | Proxmox API and, later, Kubernetes API | Only supplement information missing from the overview. |

K3s uses containerd by default; no Docker API is required for the cluster. See [K3s runtime configuration](https://docs.k3s.io/advanced). Docker hosts will receive their own adapter when actually needed.

## Initial Scope

Start with exactly one source that’s actually available, such as Proxmox. The first page displays the hostname, availability, and some meaningful metrics. Then add Prometheus and Kubernetes.

- Hosts/nodes with status and last successful update.
- CPU, RAM, and storage space, as long as the source provides them.
- Workload status and links to Grafana or Argo CD.
- Status indicators for loading, missing data, source failure, and stale data.
- Initial update interval of about 30 seconds.

## Access Model

Third-party APIs are called by the backend. The browser receives processed data and no infrastructure credentials.

Use a separate account or token with the necessary read permissions for each source. In Kubernetes, use a dedicated ServiceAccount with limited RBAC; do not use a `cluster-admin` Kubeconfig or an administrative Docker socket. The application will not initially have start, stop, or delete functions.

## Acceptance and Operation

1. A live source provides reproducible values to the overview page.
2. A source timeout does not block the entire page or other sources.
3. Outdated data is displayed along with the timestamp and error status.
4. A source failure does not automatically cause the dashboard to be marked as unhealthy; display the source status separately.
5. Credentials do not appear in browser responses or logs.
6. The App Image can be launched with a documented source configuration; synthetic sample data is available for local development.

Tests primarily verify error mapping and data mapping. Additionally, verify API contracts and permissions against the actual source; a mock does not prove functional infrastructure access.

## Future Expansion

Backup overview, power consumption, PV metrics, certificate expiration, deployment versions, and repo links. Initially maintain alert rules in the existing monitoring platform.

## Planned Repo Structure

```text
homelab-pulse-monitor/
├── README.md
├── Preparation.md
├── src/Pulse.Web/
├── tests/
├── deploy/base/
└── .gitlab-ci.yml
```

## Description for the Future README


> Homelab Pulse Monitor is a planned internal dashboard built with C# and Blazor. 
The backend aggregates selected Homelab statuses with read-only access and displays errors as well as the recency of the values. 
The first milestone integrates a real-world data source; further integrations will follow gradually.
