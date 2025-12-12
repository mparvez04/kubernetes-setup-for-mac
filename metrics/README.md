## Install Metrics Server - Monitoring Cluster Components

High Availability
Metrics Server can be installed in high availability mode directly from a YAML manifest or via the official Helm chart by setting the replicas value greater than 1. To install the latest Metrics Server release in high availability mode from the high-availability.yaml manifest, run the following command.

On Controlplane server:
1. download the high-availability.yaml manifest

```
# wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml -O high-availability.yaml
```

2. To fix the know issue below update the high-availability.yaml file

`
missing content for ca bundle \ client-ca::kube-system::extension-apiserver-authentication::requestheader-client-ca-file\ logger= unhandlederror
`

3. Get configmap information 
```
# kubectl get configmap extension-apiserver-authentication -n kube-system -o yaml
```

4. Truncated output below 
```
apiVersion: v1
data:
  client-ca-file: |
    -----BEGIN CERTIFICATE-----
kind: ConfigMap
metadata:
  name: extension-apiserver-authentication
  namespace: kube-system
  
```
5. Update the high-availability.yaml file
 to Mount configmap in Metrics Server deployment and add --requestheader-client-ca-file flag

```
      - args:
        - --requestheader-client-ca-file=/ca/client-ca-file // ADD THIS!
        - --cert-dir=/tmp
        - --secure-port=10250
        volumeMounts:
        - mountPath: /ca // ADD THIS!
          name: ca-dir

      volumes:
      - configMap: // ADD THIS!
          defaultMode: 420
          name: extension-apiserver-authentication
        name: ca-dir
```

6. Install Metrics Server

```
# kubectl apply -f high-availability.yaml
```

## Known Issue

[Incorrectly configured front-proxy certificate](https://github.com/kubernetes-sigs/metrics-server/blob/master/KNOWN_ISSUES.md#incorrectly-configured-front-proxy-certificate)