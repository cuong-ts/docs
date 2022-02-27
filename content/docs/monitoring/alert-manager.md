# Alert manager

{% code title="alertmanager.yml" %}
```bash
global:
  resolve_timeout: 1m
  slack_api_url: 'https://hooks.slack.com/services/xxx'
route:
  # When a new group of alerts is created by an incoming alert, wait at
  # least 'group_wait' to send the initial notification.
  # This way ensures that you get multiple alerts for the same group that start
  # firing shortly after another are batched together on the first
  # notification.
  group_wait: 10s

  # When the first notification was sent, wait 'group_interval' to send a batch
  # of new alerts that started firing for that group.
  group_interval: 15m

  # If an alert has successfully been sent, wait 'repeat_interval' to
  # resend them.
  repeat_interval: 15m

  # A default receiver
  receiver: "slack-warning"

  # All the above attributes are inherited by all child routes and can
  # overwritten on each.
  routes:
    - receiver: "slack-critical"
      group_wait: 10s
      match_re:
        severity: critical
      continue: true
      group_by: [alertname, severity, app]
    - receiver: "slack-warning"
      group_wait: 10s
      match_re:
        severity: warning|info
      continue: true
      group_by: [alertname, severity, app]

receivers:
  - name: "slack-critical"
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx'
        channel: 'alert-manager'
        color: '{{ template "slack.scs.color" . }}'
        title: '{{ template "slack.scs.title" . }}'
        text: '{{ template "slack.scs.text" . }}'
        send_resolved: true
        actions:
          - type: button
            text: 'Runbook :green_book:'
            url: '{{ (index .Alerts 0).Annotations.runbook_url }}'
          - type: button
            text: 'Query :mag:'
            url: '{{ (index .Alerts 0).GeneratorURL }}'
          - type: button
            text: 'Dashboard :chart_with_upwards_trend:'
            url: '{{ (index .Alerts 0).Annotations.dashboard_url }}'
          - type: button
            text: 'Silence :no_bell:'
            url: '{{ template "__alert_silence_link" . }}'
          # - type: button
          #   text: '{{ template "slack.scs.link_button_text" . }}'
          #   url: '{{ .CommonAnnotations.link_url }}'
  - name: "slack-warning"
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx'
        channel: 'alert-manager'
        color: '{{ template "slack.scs.color" . }}'
        title: '{{ template "slack.scs.title" . }}'
        text: '{{ template "slack.scs.text" . }}'
        send_resolved: false
        actions:
          - type: button
            text: 'Runbook :green_book:'
            url: '{{ (index .Alerts 0).Annotations.runbook_url }}'
          - type: button
            text: 'Query :mag:'
            url: '{{ (index .Alerts 0).GeneratorURL }}'
          - type: button
            text: 'Dashboard :chart_with_upwards_trend:'
            url: '{{ (index .Alerts 0).Annotations.dashboard_url }}'
          - type: button
            text: 'Silence :no_bell:'
            url: '{{ template "__alert_silence_link" . }}'
          # - type: button
          #   text: '{{ template "slack.scs.link_button_text" . }}'
          #   url: '{{ .CommonAnnotations.link_url }}'
templates: ['/etc/alertmanager/templates/*.tmpl']
```
{% endcode %}

Alert rules:

{% tabs %}
{% tab title="backbox" %}
```bash
groups:
- name: BlackboxGroup
  rules:
  - alert: WebDown
    expr: probe_success == 0
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "{{ $labels.instance }} is down"
      description: 'HTTP check failed'
      dashboard_url: "dashboard/d/UV2ducRnk/blackbox-exporter-http-prober?orgId=1"

  # - alert: BlackboxProbeHttpFailure
  #   expr: probe_http_status_code <= 199 OR probe_http_status_code >= 400
  #   for: 0m
  #   labels:
  #     severity: critical
  #   annotations:
  #     summary: Blackbox probe HTTP failure (instance {{ $labels.instance }})
  #     description: "HTTP status code is not 200-399\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  # - alert: BlackboxSslCertificateWillExpireSoon

  - alert: SSLCertExpired
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 7
    for: 2d
    labels:
      severity: warning
    annotations:
      summary: "SSL certificate will expire soon"
      description: "{{ $labels.instance }} - Remaining time: `{{ humanizeDuration $value }}`"
      dashboard_url: "dashboard/d/UV2ducRnk/blackbox-exporter-http-prober?orgId=1"

  - alert: SSLCertExpired
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 3
    for: 1d
    labels:
      severity: critical
    annotations:
      summary: "SSL certificate will expire soon"
      description: "{{ $labels.instance }} - Remaining time: `{{ humanizeDuration $value }}`"
      dashboard_url: "dashboard/d/UV2ducRnk/blackbox-exporter-http-prober?orgId=1"

  # - alert: BlackboxSslCertificateExpired
  #   expr: probe_ssl_earliest_cert_expiry - time() <= 0
  #   for: 0m
  #   labels:
  #     severity: critical
  #   annotations:
  #     summary: Blackbox SSL certificate expired (instance {{ $labels.instance }})
  #     description: "SSL certificate has expired already\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  
  # - alert: BlackboxProbeSlowHttp
  #   expr: avg_over_time(probe_http_duration_seconds[1m]) > 1
  #   for: 1m
  #   labels:
  #     severity: warning
  #   annotations:
  #     summary: Blackbox probe slow HTTP (instance {{ $labels.instance }})
  #     description: "HTTP request took more than 1s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```
{% endtab %}

{% tab title="node" %}
```bash
groups:
- name: node_exporter
  rules:
  - alert: OutOfMemory
    expr: 100-(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100) > 95
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "`{{ $labels.job }}` out of RAM"
      description: 'RAM used > 95% for 5mins. Current value = `{{ printf "%.2f" $value }}%`'
      dashboard_url: "dashboard/d/hb7fSE0Zz/scs-servers?orgId=1&var-job={{ $labels.job }}&var-hostname=All&var-node={{ $labels.instance }}"

  # Please add ignored mountpoints in node_exporter parameters like
  # "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|run)($|/)".
  # Same rule using "node_filesystem_free_bytes" will fire when disk fills for non-root users.
  - alert: OutOfDisk
    expr:   ( 100-(node_filesystem_avail_bytes / node_filesystem_size_bytes * 100) > 95 and node_filesystem_readonly == 0)
    for: 1h
    labels:
      severity: warning
    annotations:
      summary: "`{{ $labels.job }}` out of disk"
      description: 'Disk used > 95%. Current value `{{ printf "%.2f" $value }}%`'
      dashboard_url: "dashboard/d/hb7fSE0Zz/scs-servers?orgId=1&var-job={{ $labels.job }}&var-hostname=All&var-node={{ $labels.instance }}"

#  - alert: OutOfDisk
#     expr:   ( 100-(node_filesystem_avail_bytes / node_filesystem_size_bytes * 100) > 95 and node_filesystem_readonly == 0)
#     for: 1h
#     labels:
#       severity: critical
#     annotations:
#       summary: "<!channel> `{{ $labels.job }}` out of disk"
#       description: 'Disk used > 95%. Current value `{{ printf "%.2f" $value }}%`'
#       dashboard_url: "dashboard/d/hb7fSE0Zz/scs-servers?orgId=1&var-job={{ $labels.job }}&var-hostname=All&var-node={{ $labels.instance }}"

  - alert: HighCpuLoad
    expr: 100 - (avg by(instance,job) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 95
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: "`{{ $labels.job }}` high CPU load"
      description: 'CPU load > 95% for 5mins. Current value = `{{ printf "%.2f" $value }}%`'
      dashboard_url: "dashboard/d/hb7fSE0Zz/scs-servers?orgId=1&var-job={{ $labels.job }}&var-hostname=All&var-node={{ $labels.instance }}"

```
{% endtab %}

{% tab title="template" %}
```bash
{{ define "__alert_silence_link" -}}
    {{ .ExternalURL }}/#/silences/new?filter=%7B
    {{- range .CommonLabels.SortedPairs -}}
        {{- if ne .Name "alertname" -}}
            {{- .Name }}%3D"{{- .Value -}}"%2C%20
        {{- end -}}
    {{- end -}}
    alertname%3D"{{- .CommonLabels.alertname -}}"%7D
{{- end }}

{{ define "__alert_severity" -}}
    {{- if eq .CommonLabels.severity "critical" -}}
    *Severity:* <!channel> `Critical`
    {{- else if eq .CommonLabels.severity "warning" -}}
    *Severity:* `Warning`
    {{- else if eq .CommonLabels.severity "info" -}}
    *Severity:* `Info`
    {{- else -}}
    *Severity:* :question: {{ .CommonLabels.severity }}
    {{- end }}
{{- end }}

{{ define "slack.scs.title" -}}
  [{{ .Status | toUpper -}}
  {{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{- end -}}
  ] {{ .CommonLabels.alertname }}
{{- end }}


{{ define "slack.scs.text" -}}

    {{ template "__alert_severity" . }}
    {{- if (index .Alerts 0).Annotations.summary }}
    {{- "\n" -}}
    *Summary:* {{ (index .Alerts 0).Annotations.summary }}
    {{- end }}

    {{ range .Alerts }}

        {{- if .Annotations.description }}
        {{- "\n" -}}
        {{ .Annotations.description }}
        {{- "\n" -}}
        {{- end }}
        {{- if .Annotations.message }}
        {{- "\n" -}}
        {{ .Annotations.message }}
        {{- "\n" -}}
        {{- end }}

    {{- end }}

{{- end }}

{{ define "slack.scs.color" -}}
    {{ if eq .Status "firing" -}}
        {{ if eq .CommonLabels.severity "warning" -}}
            warning
        {{- else if eq .CommonLabels.severity "critical" -}}
            danger
        {{- else -}}
            #439FE0
        {{- end -}}
    {{ else -}}
    good
    {{- end }}
{{- end }}
```
{% endtab %}
{% endtabs %}



