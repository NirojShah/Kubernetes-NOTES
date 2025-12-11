# Kubernetes NetworkPolicy Explanation (Line by Line)

``` yaml
apiVersion: networking.k8s.io/v1
```

Specifies the API version used by Kubernetes for NetworkPolicy objects.

``` yaml
kind: NetworkPolicy
```

Defines the type of Kubernetes resource---here it is a NetworkPolicy.

``` yaml
metadata:
  name: test-network-policy
  namespace: default
```

-   **name**: The name of the NetworkPolicy.
-   **namespace**: It will be created in the `default` namespace.

------------------------------------------------------------------------

# Spec Section

``` yaml
spec:
```

Contains all rules for the NetworkPolicy.

------------------------------------------------------------------------

## Pod Selector

``` yaml
  podSelector:
    matchLabels:
      role: db
```

Selects the target pods. This policy applies to all pods with the label
`role=db` in the default namespace.

------------------------------------------------------------------------

## Policy Types

``` yaml
  policyTypes:
  - Ingress
  - Egress
```

-   **Ingress**: Controls incoming traffic.
-   **Egress**: Controls outgoing traffic.

------------------------------------------------------------------------

# Ingress Rules (Incoming Traffic)

``` yaml
  ingress:
  - from:
```

Defines who is allowed to send traffic to the selected DB pods.

### Allow traffic from IP range

``` yaml
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
```

-   Allows traffic from `172.17.0.0/16`
-   Except from IPs in `172.17.1.0/24`

### Allow traffic from namespaces matching label

``` yaml
    - namespaceSelector:
        matchLabels:
          project: myproject
```

Allows incoming traffic from any pod inside a namespace labeled
`project=myproject`.

### Allow traffic from certain pods

``` yaml
    - podSelector:
        matchLabels:
          role: frontend
```

Allows traffic from pods labeled `role=frontend` in the **same
namespace** (`default`).

### Allowed ingress ports

``` yaml
    ports:
    - protocol: TCP
      port: 6379
```

Ingress traffic is allowed only on **TCP port 6379**.

------------------------------------------------------------------------

# Egress Rules (Outgoing Traffic)

``` yaml
  egress:
  - to:
```

Defines where DB pods can send outgoing traffic.

### Allowed egress IP range

``` yaml
    - ipBlock:
        cidr: 10.0.0.0/24
```

The DB pods can send traffic only to IPs in `10.0.0.0/24`.

### Allowed egress ports

``` yaml
    ports:
    - protocol: TCP
      port: 5978
```

Outgoing traffic is allowed only on **TCP port 5978**.

------------------------------------------------------------------------

# Summary

### Ingress Allowed From

-   IP range `172.17.0.0/16` **except** `172.17.1.0/24`
-   Any namespace with label `project=myproject`
-   Pods labeled `role=frontend`
-   **Port 6379 (TCP)** only

### Egress Allowed To

-   IP range `10.0.0.0/24`
-   **Port 5978 (TCP)** only

Everything else is denied by default once a NetworkPolicy exists.
