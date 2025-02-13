# Cloud SQL proxy health checks

Kubernetes supports three types of health checks.
1. Startup probes determine whether a container is done starting up. As soon as this probe succeeds, Kubernetes switches over to using liveness and readiness probing.
2. Liveness probes determine whether a container is healthy. When this probe is unsuccessful, the container is restarted.
3. Readiness probes determine whether a container can serve new traffic. When this probe fails, Kubernetes will wait to send requests to the container.

## Running Cloud SQL proxy with health checks in Kubernetes
1. Configure your Cloud SQL proxy container to include health check probes. 
    > [proxy_with_http_health_check.yaml](proxy_with_http_health_check.yaml#L77-L111)
     ```yaml
        # Recommended configurations for health check probes.
        # Probe parameters can be adjusted to best fit the requirements of your application.
        # For details, see https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
        livenessProbe:
          httpGet:
            path: /liveness
            port: 8090
          # Number of seconds after the container has started before the first probe is scheduled. Defaults to 0.
          # Not necessary when the startup probe is in use.
          initialDelaySeconds: 0
          # Frequency of the probe. Defaults to 10.
          periodSeconds: 10
          # Number of seconds after which the probe times out. Defaults to 1.
          timeoutSeconds: 5
          # Number of times the probe is allowed to fail before the transition from healthy to failure state. 
          # Defaults to 3.
          failureThreshold: 1
        readinessProbe:
          httpGet:
            path: /liveness
            port: 8090
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 5
          # Number of times the probe must report success to transition from failure to healthy state.
          # Defaults to 1 for readiness probe.
          successThreshold: 1
          failureThreshold: 1
        startupProbe:
          httpGet:
            path: /startup
            port: 8090
          periodSeconds: 1
          timeoutSeconds: 5
          failureThreshold: 20
     ```

2. Add `-use_http_health_check` and `-health-check-port` (optional) to your proxy container configuration under `command: `.
    > [proxy_with_http_health_check.yaml](proxy_with_http_health_check.yaml#L39-L55)
    ```yaml
        command:
          - "/cloud_sql_proxy"

          # If connecting from a VPC-native GKE cluster, you can use the
          # following flag to have the proxy connect over private IP
          # - "-ip_address_types=PRIVATE"

          # Replace DB_PORT with the port the proxy should listen on
          # Defaults: MySQL: 3306, Postgres: 5432, SQLServer: 1433
          - "-instances=<INSTANCE_CONNECTION_NAME>=tcp:<DB_PORT>"
          # Enables HTTP health checks.
          - "-use_http_health_check"
          # Specifies the health check server port.
          # Defaults to 8090.
          - "-health_check_port=<YOUR-HEALTH-CHECK-PORT>"
          # This flag specifies where the service account key can be found
          - "-credential_file=/secrets/service_account.json"
    ```

