# kube-oidc-proxy Helm Chart

A Helm Chart for an OpenID Connect proxy for the Kubernetes apiserver to authenticate and impersonate users. Ideal for situations where cloud providers do not give access to configure the OpenID Connect features of the Kubernetes apiserver, such as Microsoft Azure Kubernetes Service.

## Prerequisites

Kubernetes cluster-admin privileges are needed to install this Helm Chart as it creates a ClusterRole and ClusterRoleBinding. The ServiceAccount will be bound to the Deployment and the Deployment runs without root permissions as user nobody and group nobody.

## Install the Helm Chart

```shell
helm upgrade kube-oidc-proxy oci://registry.spreitzer.ch/charts/kube-oidc-proxy --install --create-namespace --namespace kube-oidc-proxy --values custom_values.yml
```

## Helm Chart Configuration Examples

Azure Kubernetes Service with ingress-nginx and CertManager ACME

```yaml
oidc:
  issuer: https://id.example.com/auth/realms/master
  clientID: kubernetes
ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: HTTPS
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/preserve-trailing-slash: "true"
    cert-manager.io/cluster-issuer: letsencrypt-production
  hosts:
    - k.mycluster.k8s.example.com
```

Generic Example

```yaml
oidc:
  issuer: https://id.example.com/auth/realms/master
  clientID: kubernetes
ingress:
  enabled: true
  hosts:
    - k.mycluster.k8s.example.com
```

kubeconfig Example with [kube-login](https://github.com/int128/kubelogin)

```yaml
apiVersion: v1
clusters:
- cluster:
    server: https://k.mycluster.k8s.example.com:443
  name: mycluster
contexts:
- context:
    cluster: mycluster
    user: oidc
  name: mylcuster
current-context: mycluster
kind: Config
users:
- name: oidc
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: kubectl
      args:
      - oidc-login
      - get-token
      - --oidc-issuer-url=https://id.example.com/auth/realms/master
      - --oidc-client-id=kubernetes
      env: null
      interactiveMode: IfAvailable
      provideClusterInfo: false
```

## Remarks

`kube-oidc-proxy` does not support setting an OIDC client secret. This Helm Chart enforces using TLS on the ingress, if enabled.

## Helm Chart Values

| Key | Description | Default |
|---|---|---|
| `oidc.issuer` | The OpenID Connect issuer URL that houses the `/.well-known/openid-configuration` file. | `https://id.example.com/auth/realms/master` |
| `oidc.clientID` | The OpenID Connect client ID. | `kubernetes` |
| `oidc.usernameClaim` | The OpenID Connect username claim. | `preferred_username` |
| `oidc.groupsClaim` | The OpenID Connect groups claim. | `groups` |
| `oidc.usernamePrefix` | The OpenID Connect username prefix. | `oidc:` |
| `oidc.groupsPrefix` | The OpenID Connect groups prefix. | `oidc:` |
| `oidc.requiredClaims` | The OpenID Connect claims required to authenticate. Array of strings in form of `claim=value` | `[]` |
| `deployment.image.name` | The container image to use. | `docker.io/tremolosecurity/kube-oidc-proxy:latest` |
| `deployment.image.pullPolicy` | The Kubernetes pull policy to use. | `Always` |
| `deployment.replicas` | The amount of parallel replicas. | `2` |
| `deployment.strategy` | The strategy to rollout new containers. | `RollingUpdate` |
| `deployment.resources` | The resource requests and limits. | See [values.yaml](/values.yaml) |
| `service.type` | The Kubernetes service type. | `ClusterIP` |
| `service.annotations` | The Kubernetes service annotations. | (none) |
| `service.externalTrafficPolicy` | The Kubernetes service external traffic policy. | (none) |
| `ingress.enabled` | Enable Kubernetes ingress. | `false` |
| `ingress.annotations` | The Kubernetes ingress annotations. | (none) |
| `ingress.hosts` | The Kubernetes ingress hosts. | `['api.kubernetes.example.com']` |
| `ingress.tlsSecret` | The Kubernetes ingress to use, if supplied. | (none) |

# References

* https://git.spreitzer.ch/helm/kube-oidc-proxy
* https://github.com/int128/kubelogin
* https://www.jetstack.io/blog/kube-oidc-proxy/
* https://github.com/jetstack/kube-oidc-proxy
* https://github.com/jetstack/kube-oidc-proxy/issues/202#issuecomment-1061826590
* https://github.com/tremolosecurity/kube-oidc-proxy
* https://www.tremolosecurity.com/post/updating-kube-oidc-proxy
* https://github.com/Azure/AKS/issues/2861
* https://github.com/Azure/AKS/issues/2861#issuecomment-1433382599
* https://github.com/Azure/AKS/issues/2861#issuecomment-1527557548
