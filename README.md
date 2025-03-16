# CloudNativePG

## Kubernetes Section ⚓

Integrating Kubernetes into this project enables efficient deployment and orchestration of microservices. Here are the steps to set up a local test environment using **kind** (Kubernetes IN Docker) 🐳 and to deploy a highly available PostgreSQL database with **CloudNativePG** 🐘.

### Creating a Kubernetes Cluster with kind 🔨

**kind** is a tool that lets you create local Kubernetes clusters using Docker as the container runtime. It’s perfect for quickly testing deployments in a simple environment. ⚡

```sh
cat <<EOF | kind create cluster -n my-cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
  - containerPort: 5432
    hostPort: 5432
    protocol: TCP
EOF
```

## Create nginx controller

[Documentation](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx)

```sh
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

```sh
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## Deploying CloudNativePG 🐘

[Documentation](https://cloudnative-pg.io/documentation/current/installation_upgrade/)

**CloudNativePG** is a solution that simplifies managing PostgreSQL databases in a Kubernetes environment, providing high availability features. 🚀 This sets up the necessary resources to run PostgreSQL reliably and efficiently. 🛡️

```sh
kubectl apply --server-side -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.25/releases/cnpg-1.25.1.yaml
```

### Deploy it 🚀

```sh
k apply -k k8s/base .
```

## Install kubectl plugin

```sh
curl -sSfL \
  https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \
  sudo sh -s -- -b /usr/local/bin
```

## CLI Documentation

```txt
A plugin to manage your CloudNativePG clusters

Usage:
  kubectl cnpg [command]

Operator-level administration
  install      CloudNativePG installation-related commands

Troubleshooting
  logs         Logging utilities
  report       Report on the operator or a cluster for troubleshooting

Cluster administration
  destroy      Destroy the instance named CLUSTER-INSTANCE with the associated PVC
  fencing      Fencing related commands
  hibernate    Hibernation related commands
  maintenance  Sets or removes maintenance mode from clusters
  promote      Promote the instance named CLUSTER-INSTANCE to primary
  reload       Reload a cluster
  restart      Restart a cluster or a single instance in a cluster

Database administration
  backup       Request an on-demand backup for a PostgreSQL Cluster
  certificate  Create a client certificate to connect to PostgreSQL using TLS and Certificate authentication
  publication  Logical publication management commands
  snapshot     DEPRECATED (use `backup -m volumeSnapshot` instead)
  status       Get the status of a PostgreSQL cluster
  subscription Logical subscription management commands

Miscellaneous
  fio          Creates a fio deployment, pvc and configmap
  pgadmin4     Creates a pgAdmin deployment
  pgbench      Creates a pgbench job
  psql         Start a psql session targeting a CloudNativePG cluster

Additional Commands:
  completion   Generate the autocompletion script for the specified shell
  help         Help about any command
  version      Prints version, commit sha and date of the build

Flags:
      --as string                         Username to impersonate for the operation. User could be a regular user or a service account in a namespace.
      --as-group stringArray              Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --as-uid string                     UID to impersonate for the operation.
      --cache-dir string                  Default cache directory (default "/home/fabien/.kube/cache")
      --certificate-authority string      Path to a cert file for the certificate authority
      --client-certificate string         Path to a client certificate file for TLS
      --client-key string                 Path to a client key file for TLS
      --cluster string                    The name of the kubeconfig cluster to use
      --context string                    The name of the kubeconfig context to use
      --disable-compression               If true, opt-out of response compression for all requests to the server
  -h, --help                              help for kubectl cnpg
      --insecure-skip-tls-verify          If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string                 Path to the kubeconfig file to use for CLI requests.
      --log-destination string            where the log stream will be written
      --log-field-level string            JSON log field to report severity in (default: level)
      --log-field-timestamp string        JSON log field to report timestamp in (default: ts)
      --log-level string                  the desired log level, one of error, info, debug and trace (default "info")
  -n, --namespace string                  If present, the namespace scope for this CLI request
      --request-timeout string            The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
  -s, --server string                     The address and port of the Kubernetes API server
      --tls-server-name string            Server name to use for server certificate validation. If it is not provided, the hostname used to contact the server is used
      --token string                      Bearer token for authentication to the API server
      --user string                       The name of the kubeconfig user to use
      --zap-devel                         Development Mode defaults(encoder=consoleEncoder,logLevel=Debug,stackTraceLevel=Warn). Production Mode defaults(encoder=jsonEncoder,logLevel=Info,stackTraceLevel=Error)
      --zap-encoder encoder               Zap log encoding (one of 'json' or 'console')
      --zap-log-level level               Zap Level to configure the verbosity of logging. Can be one of 'debug', 'info', 'error', or any integer value > 0 which corresponds to custom debug levels of increasing verbosity
      --zap-stacktrace-level level        Zap Level at and above which stacktraces are captured (one of 'info', 'error', 'panic').
      --zap-time-encoding time-encoding   Zap time encoding (one of 'epoch', 'millis', 'nano', 'iso8601', 'rfc3339' or 'rfc3339nano'). Defaults to 'epoch'.

Use "kubectl cnpg [command] --help" for more information about a command.
```

## Documentations

- [Backup](https://cloudnative-pg.io/documentation/current/backup/)
- [Backup on object stores](https://cloudnative-pg.io/documentation/current/backup_barmanobjectstore/)
- [backup_volumesnapshot](https://cloudnative-pg.io/documentation/current/backup_volumesnapshot/)
- [object_stores](https://cloudnative-pg.io/documentation/current/appendixes/object_stores/)
- [pod-affinity-and-anti-affinity](https://cloudnative-pg.io/documentation/current/scheduling/#pod-affinity-and-anti-affinity)
- [Monitoring](https://cloudnative-pg.io/documentation/current/monitoring/)
- [Dashboard Grafana](https://github.com/cloudnative-pg/grafana-dashboards)

## Troobleshooting

```txt
k cnpg backup cluster-example -m volumeSnapshot

Error: admission webhook "vbackup.cnpg.io" denied the request: Backup.postgresql.cnpg.io "cluster-example-20250316160747" is invalid: spec.method: Invalid value: "volumeSnapshot": Cannot use volumeSnapshot backup method due to missing VolumeSnapshot CRD. If you installed the CRD after having started the operator, please restart it to enable VolumeSnapshot support
```
