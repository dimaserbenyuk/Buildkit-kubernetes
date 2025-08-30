# Buildkit-kubernetes
Buildkit-kubernetes

```shell
brew install buildkit
```

1. without ssl

apply

```shell
kubectl apply -f buildkit-no-ssl-arm.yml
```

port-forward

```shell
kubectl port-forward service/buildkitd-arm 1234:1234 -n buildkit
```
```shell
Forwarding from 127.0.0.1:1234 -> 1234
Forwarding from [::1]:1234 -> 1234
Handling connection for 1234
```

check workers

```shell
buildctl --addr tcp://127.0.0.1:1234 debug workers
```
output, all good 

```shell
ID				PLATFORMS
vvgzvgoj6iexoir7096hz07fm	linux/arm64,linux/amd64,linux/amd64/v2,linux/riscv64,linux/ppc64le,linux/s390x,linux/386,linux/arm/v7,linux/arm/v6

```
info

```shell
buildctl --addr tcp://127.0.0.1:1234 debug info
```

buildctl --addr tcp://127.0.0.1:1234 du

output

```shell
BuildKit: github.com/moby/buildkit v0.23.0 cc8ff80e5733eb0a0347176009232d6e40752f7f
```

create remote builder

```shell
docker buildx create --name remote --driver remote tcp://127.0.0.1:1234 --use
```
output
```shell
remote
```

check buildx

```shell
docker buildx ls
```

output

```shell
NAME/NODE           DRIVER/ENDPOINT                                                                                     STATUS     BUILDKIT   PLATFORMS
kube                kubernetes
 \_ kube0            \_ kubernetes:///kube?deployment=buildkit-132097b2-0aa3-422f-a13d-49c728de008d-f5389&kubeconfig=   inactive
remote*             remote
 \_ remote0          \_ tcp://127.0.0.1:1234                                                                            running    v0.23.0    linux/amd64 (+2), linux/arm64, linux/arm (+2), linux/ppc64le, (3 more)
default             docker
 \_ default          \_ default                                                                                         running    v0.21.0    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
desktop-linux       docker
 \_ desktop-linux    \_ desktop-linux                                                                                   running    v0.21.0    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
```

buildx

```shell
remote*             remote
 \_ remote0          \_ tcp://127.0.0.1:1234
```

docker run without cache

```shell
docker buildx build \
    --platform linux/arm64 \
    --builder=remote \
    -t django:latest \
    -f Dockerfile . \
    --progress=plain \
    --load
```

<details>
<summary>Docker running logs</summary>
<br>
building with "remote" instance using remote driver.
<br><br>
<pre>
#0 building with "remote" instance using remote driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 1.92kB done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/python:3.13-slim
#2 DONE 1.0s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [1/9] FROM docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81
#4 resolve docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81 done
#4 DONE 0.0s

#5 [2/9] WORKDIR /usr/src/app
#5 CACHED

#6 [internal] load build context
#6 transferring context: 1.56kB done
#6 DONE 0.0s

#7 [3/9] RUN apt-get update && apt-get install -y --no-install-recommends     build-essential     libpq-dev     gcc     libcairo2     libpango-1.0-0     libpangocairo-1.0-0     libgdk-pixbuf-2.0-0     libffi-dev     shared-mime-info     libxml2     libxslt1.1     libjpeg-dev     libglib2.0-0     fonts-liberation  && apt-get clean && rm -rf /var/lib/apt/lists/*
#7 0.668 Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
#7 1.048 Get:2 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
#7 1.196 Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
#7 1.326 Get:4 http://deb.debian.org/debian bookworm/main arm64 Packages [8693 kB]
#7 2.103 Get:5 http://deb.debian.org/debian bookworm-updates/main arm64 Packages [756 B]
#7 2.227 Get:6 http://deb.debian.org/debian-security bookworm-security/main arm64 Packages [264 kB]
#7 2.609 Fetched 9213 kB in 2s (3704 kB/s)
...
...
#13 DONE 0.1s

#14 exporting to oci image format
#14 exporting layers
#14 exporting layers 77.4s done
#14 exporting manifest sha256:212eaf6eacaaf8eb5e90ca10dc048445ece74a4326d4e8f204401f2e6aae1193
#14 exporting manifest sha256:212eaf6eacaaf8eb5e90ca10dc048445ece74a4326d4e8f204401f2e6aae1193 done
#14 exporting config sha256:bdaea0e8ef6da32d24cdd1eed008cc4c769a022490edf7dacd1dc61bccaf7c9e done
#14 sending tarball
#14 ...

#15 importing to docker
#15 DONE 0.0s

#14 exporting to oci image format
#14 sending tarball 4.7s done
#14 DONE 82.1s
</pre>
</details>

connect to pod check cache

```shell
ls -lah /var/lib/buildkit
```
output

```shell
/ 
total 356K   
drwxr-xr-x    3 root     root        4.0K Jun 24 10:20 .
drwxr-xr-x    1 root     root        4.0K Jun 24 10:20 ..
-rw-------    1 root     root           0 Jun 24 10:20 buildkitd.lock
-rw-------    1 root     root      128.0K Jun 24 10:36 cache.db
-rw-------    1 root     root      256.0K Jun 24 10:38 history.db
drwx------    6 root     root        4.0K Jun 18 15:15 runc-overlayfs
```

run again 

```
#7 [6/9] COPY manage.py .
#7 CACHED
```
`CACHED` - means that this step is taken from the cache and not executed again

```shell
docker buildx build \
    --platform linux/arm64 \
    --builder=remote \
    -t django:latest \
    -f Dockerfile . \
    --progress=plain \
    --load
```

```logs    
#0 building with "remote" instance using remote driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 1.92kB done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/python:3.13-slim
#2 ...

#3 [auth] library/python:pull token for registry-1.docker.io
#3 DONE 0.0s

#2 [internal] load metadata for docker.io/library/python:3.13-slim
#2 DONE 1.4s

#4 [internal] load .dockerignore
#4 transferring context: 2B done
#4 DONE 0.0s

#5 [1/9] FROM docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81
#5 resolve docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81 done
#5 DONE 0.0s

#6 [internal] load build context
#6 transferring context: 1.11kB done
#6 DONE 0.0s

#7 [6/9] COPY manage.py .
#7 CACHED

#8 [4/9] COPY requirements.txt .
#8 CACHED

#9 [2/9] WORKDIR /usr/src/app
#9 CACHED

#10 [3/9] RUN apt-get update && apt-get install -y --no-install-recommends     build-essential     libpq-dev     gcc     libcairo2     libpango-1.0-0     libpangocairo-1.0-0     libgdk-pixbuf-2.0-0     libffi-dev     shared-mime-info     libxml2     libxslt1.1     libjpeg-dev     libglib2.0-0     fonts-liberation  && apt-get clean && rm -rf /var/lib/apt/lists/*
#10 CACHED

#11 [5/9] RUN python3 -m venv /opt/venv &&     . /opt/venv/bin/activate &&     pip install --upgrade pip &&     pip install -r requirements.txt
#11 CACHED

#12 [7/9] COPY backend/ backend/
#12 CACHED

#13 [8/9] COPY entrypoint.sh /entrypoint.sh
#13 CACHED

#14 [9/9] RUN chmod +x /entrypoint.sh &&     mkdir -p /usr/src/app/.cache/fontconfig &&     groupadd -r appgroup -g 1000 &&     useradd -r -u 1000 -g appgroup appuser &&     chown -R appuser:appgroup /usr/src/app
#14 CACHED

#15 exporting to oci image format
#15 exporting layers done
#15 exporting manifest sha256:212eaf6eacaaf8eb5e90ca10dc048445ece74a4326d4e8f204401f2e6aae1193 done
#15 exporting config sha256:bdaea0e8ef6da32d24cdd1eed008cc4c769a022490edf7dacd1dc61bccaf7c9e done
#15 sending tarball
#15 sending tarball 2.8s done
#15 DONE 2.8s

#16 importing to docker
#16 DONE 0.0s
```


example 2

```shell
helm repo add jetstack https://charts.jetstack.io
"jetstack" has been added to your repositories
```

```shell
helm search repo jetstack
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                                       
jetstack/cert-manager                   v1.18.1         v1.18.1         A Helm chart for cert-manager                     
jetstack/cert-manager-approver-policy   v0.21.0         v0.21.0         approver-policy is a CertificateRequest approve...
jetstack/cert-manager-csi-driver        v0.10.3         v0.10.3         cert-manager csi-driver enables issuing secretl...
jetstack/cert-manager-csi-driver-spiffe v0.9.1          v0.9.1          csi-driver-spiffe is a Kubernetes CSI plugin wh...
jetstack/cert-manager-google-cas-issuer v0.10.0         v0.10.0         A Helm chart for jetstack/google-cas-issuer       
jetstack/cert-manager-istio-csr         v0.14.1         v0.14.1         istio-csr enables the use of cert-manager for i...
jetstack/cert-manager-trust             v0.2.1          v0.2.0          DEPRECATED: The old name for trust-manager. Use...
jetstack/finops-dashboards              v0.0.5          0.0.5           A Helm chart for Kubernetes                       
jetstack/finops-policies                v0.0.6          v0.0.6          A Helm chart for Kubernetes                       
jetstack/finops-stack                   v0.0.5          0.0.3           A FinOps Stack for Kubernetes                     
jetstack/trust-manager                  v0.17.1         v0.17.1         trust-manager is the easiest way to manage TLS ...
jetstack/version-checker                v0.9.3          v0.9.3          A Helm chart for version-checker 
```

```shell
helm pull jetstack/cert-manager --version 1.18.1 --untar
```

```shell
helm install cert-manager ./cert-manager --values cert-manager/values.yaml
```
```shell
NAME: cert-manager
LAST DEPLOYED: Tue Jun 24 13:18:32 2025
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
```

An example of a chain of trust

```table
CLIENT ---> trusts ---> [ ca.crt ]
                            ↓
               signs   ┌──────────────┐
                       │ tls.crt      │   <-- server cert (BuildKit)
                       │ tls.key      │   <-- private key
                       └──────────────┘
```

How Certificate Signing Works — Step by Step
1. Root CA (Certificate Authority)
   
This is a self-signed certificate. It:

- Is issued once;

- Stores the private key securely;

- Is used to sign all subsequent certificates;

- Acts as the trust anchor: all clients that trust this CA will automatically trust any certificates signed by it.

In your case, the root CA is defined like this:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ca-root
spec:
  isCA: true
  secretName: buildkit-ca-secret
  issuerRef:
    name: selfsigned-ca  # <-- self-signed issuer
```
2. Issuer (Issues Real Certificates)
An Issuer (or ClusterIssuer) references the root CA and acts as a signing authority for other certificates:

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: buildkit-ca-issuer
spec:
  ca:
    secretName: buildkit-ca-secret
```

This Issuer uses the private key stored in buildkit-ca-secret to sign certificate requests.

3. Server or Client Certificates (Signed)

A real certificate (e.g. for a server or client like BuildKit) might look like this:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: buildkitd-server
spec:
  secretName: buildkitd-server-tls
  issuerRef:
    name: buildkit-ca-issuer  # <-- who signs this cert
```

cert-manager will:

- Generate a Certificate Signing Request (CSR);

- Pass it to the buildkit-ca-issuer;

- The issuer signs it using the root CA;

- The resulting Kubernetes secret (buildkitd-server-tls) will include:

`tls.crt` – the signed certificate;

`tls.key` – the private key;

`ca.crt` – (optional) the CA certificate / chain.

| Element              | Role                         | Purpose                                          |
| -------------------- | ---------------------------- | ------------------------------------------------ |
| `ca-root`            | Self-signed Root CA          | The primary trusted authority (trust anchor)     |
| `buildkit-ca-issuer` | Issuer based on Root CA      | Signs all server/client certificates             |
| `buildkitd-server`   | TLS certificate for BuildKit | Used to secure server-side TLS connections       |
| `buildctl-client`    | mTLS client certificate      | Used to authenticate client connections via mTLS |



Check the current expiration date of the root CA:

```shell
kubectl get secret buildkit-ca-secret -n buildkit -o jsonpath='{.data.ca\.crt}' | base64 -d | openssl x509 -noout -text
```

output

```shell
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            ba:12:71:a0:32:cb:5b:8b:4a:f6:f3:96:bb:56:51:ac
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=buildkit-ca
        Validity
            Not Before: Jun 24 11:25:17 2025 GMT
            Not After : Jun 22 11:25:17 2035 GMT
        Subject: CN=buildkit-ca
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d7:7a:c4:
.......
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier:
                40:F5:F1:39:67:74:A0:31:DD
    Signature Algorithm: sha256WithRSAEncryption
```

CA 10 year

```shell
kubectl get secret buildkit-ca-secret -n buildkit -o jsonpath='{.data.ca\.crt}' | base64 -d | openssl x509 -noout -dates
```

output

```shell
notBefore=Jun 24 11:25:17 2025 GMT
notAfter=Jun 22 11:25:17 2035 GMT
```

check tls 1 year

```shell
kubectl get secret buildkitd-server-tls -n buildkit -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates
```

```shell
notBefore=Jun 24 11:25:17 2025 GMT
notAfter=Jun 24 11:25:17 2026 GMT
```

```shell
kubectl get secret buildkitd-server-tls -n buildkit -o json | jq -r '.data["tls.crt"]' | base64 -d > cert.pem
kubectl get secret buildkitd-server-tls -n buildkit -o json | jq -r '.data["tls.key"]' | base64 -d > key.pem
kubectl get secret buildkit-ca-secret -n buildkit -o json | jq -r '.data["ca.crt"]' | base64 -d > ca.pem
```

```shell
kubectl create secret generic buildkit-daemon-certs \
  --namespace buildkit \
  --from-file=ca.pem=ca.pem \
  --from-file=cert.pem=cert.pem \
  --from-file=key.pem=key.pem
```

```shell
 buildctl \
  --addr tcp://127.0.0.1:1234 \
  --tlscacert ca.pem \
  --tlscert cert.pem \
  --tlskey key.pem \
  debug workers

error: failed to list workers: Unavailable: connection error: desc = "transport: authentication handshake failed: tls: failed to verify certificate: x509: cannot validate certificate for 127.0.0.1 because it doesn't contain any IP SANs"
```

fix add 127.0.0.1

```shell
echo "127.0.0.1 buildkitd.example.com" | sudo tee -a /etc/hosts
```

```shell
buildctl \
  --addr tcp://buildkitd.example.com:1234 \
  --tlscacert ca.pem \
  --tlscert cert.pem \
  --tlskey key.pem \
  --tlsservername buildkitd.example.com \
  debug workers
```

```shell
ID				PLATFORMS
dptgp7yorzm4afymsmmpieysg	linux/arm64,linux/amd64,linux/amd64/v2,linux/riscv64,linux/ppc64le,linux/s390x,linux/386,linux/arm/v7,linux/arm/v6
```

run with cache

```shell
buildctl \
  --addr tcp://buildkitd.example.com:1234 \
  --tlscacert ca.pem \
  --tlscert cert.pem \
  --tlskey key.pem \
  build \
  --frontend dockerfile.v0 \
  --local context=. \
  --local dockerfile=. \
  --opt filename=Dockerfile.cache \
  --progress=plain \
  --import-cache type=registry,ref=docker.io/serbenyuk/buildkit:cache \
  --export-cache type=registry,ref=docker.io/serbenyuk/buildkit:cache,mode=max \
  --output type=image,name=docker.io/serbenyuk/buildkit:latest,push=true
```


<details>
<summary>Docker running logs with cache</summary>
<br>
building with "remote" instance using remote driver.
<br><br>
<pre>
#1 [internal] load build definition from Dockerfile.cache
#1 transferring dockerfile: 1.46kB done
#1 DONE 0.0s

#2 resolve image config for docker-image://docker.io/docker/dockerfile:1.4
#2 DONE 0.7s

#3 docker-image://docker.io/docker/dockerfile:1.4@sha256:9ba7531bd80fb0a858632727cf7a112fbfd19b17e94c4e84ced81e24ef1a0dbc
#3 resolve docker.io/docker/dockerfile:1.4@sha256:9ba7531bd80fb0a858632727cf7a112fbfd19b17e94c4e84ced81e24ef1a0dbc done
#3 CACHED

#4 [internal] load .dockerignore
#4 transferring context: 2B done
#4 DONE 0.0s

#5 [internal] load metadata for docker.io/library/python:3.13-slim
#5 DONE 0.2s

#6 [internal] load build context
#6 DONE 0.0s

#7 [stage-0  1/10] FROM docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81
#7 resolve docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81 done
#7 DONE 0.0s

#8 importing cache manifest from docker.io/serbenyuk/buildkit:cache
#8 inferred cache manifest type: application/vnd.oci.image.manifest.v1+json done
#8 DONE 0.7s

#6 [internal] load build context
#6 transferring context: 594B done
#6 DONE 0.0s

#9 [stage-0  9/10] COPY entrypoint.sh /entrypoint.sh
#9 CACHED

#10 [stage-0  3/10] RUN --mount=type=cache,target=/var/cache/apt     apt-get update && apt-get install -y --no-install-recommends     build-essential     libpq-dev     gcc     libcairo2     libpango-1.0-0     libpangocairo-1.0-0     libgdk-pixbuf-2.0-0     libffi-dev     shared-mime-info     libxml2     libxslt1.1     libjpeg-dev     libglib2.0-0     fonts-liberation  && apt-get clean && rm -rf /var/lib/apt/lists/*
#10 CACHED

#11 [stage-0  8/10] COPY backend/ backend/
#11 CACHED

#12 [stage-0  4/10] COPY requirements.txt .
#12 CACHED

#13 [stage-0  6/10] RUN --mount=type=cache,target=/root/.cache/pip     . /opt/venv/bin/activate &&     pip install --upgrade pip &&     pip install -r requirements.txt
#13 CACHED

#14 [stage-0  7/10] COPY manage.py .
#14 CACHED

#15 [stage-0  2/10] WORKDIR /usr/src/app
#15 CACHED

#16 [stage-0  5/10] RUN python3 -m venv /opt/venv
#16 CACHED

#17 [stage-0 10/10] RUN chmod +x /entrypoint.sh &&     mkdir -p /usr/src/app/.cache/fontconfig &&     groupadd -r appgroup -g 1000 &&     useradd -r -u 1000 -g appgroup appuser &&     chown -R appuser:appgroup /usr/src/app
#17 CACHED

#18 [auth] serbenyuk/buildkit:pull,push token for registry-1.docker.io
#18 DONE 0.0s

#19 exporting to image
#19 exporting layers done
#19 exporting manifest sha256:5ce779671285edecbc413269d58fdd80bc4f478223e977ceb2ee5734c78555b3 done
#19 exporting config sha256:0596a95a49c3fd5da56f3dbac62c113e25221fe14ce881cd82168f4e4ee21685 done
#19 pushing layers
#19 pushing layers 1.9s done
#19 pushing manifest for docker.io/serbenyuk/buildkit:latest@sha256:5ce779671285edecbc413269d58fdd80bc4f478223e977ceb2ee5734c78555b3
#19 pushing manifest for docker.io/serbenyuk/buildkit:latest@sha256:5ce779671285edecbc413269d58fdd80bc4f478223e977ceb2ee5734c78555b3 0.2s done
#19 DONE 2.1s

#20 exporting cache to registry
#20 preparing build cache for export
#20 writing layer sha256:00cb57eb3fd855ce60912d767e334f0ba7cb83fa5a9193015314eb20c37fa1df
#20 writing layer sha256:00cb57eb3fd855ce60912d767e334f0ba7cb83fa5a9193015314eb20c37fa1df 0.3s done
#20 writing layer sha256:071e771a343ded6c4fa4efa0fafd47a34acf5490fafaf883243520b54b446e61
#20 writing layer sha256:071e771a343ded6c4fa4efa0fafd47a34acf5490fafaf883243520b54b446e61 0.3s done
#20 writing layer sha256:268ab4f9d69628c88961a2e7328987521ba9cb826c91598abc9744c243b148a2
#20 writing layer sha256:268ab4f9d69628c88961a2e7328987521ba9cb826c91598abc9744c243b148a2 0.3s done
#20 writing layer sha256:34ef2a75627f6089e01995bfd3b3786509bbdc7cfb4dbc804b642e195340dbc9
#20 writing layer sha256:34ef2a75627f6089e01995bfd3b3786509bbdc7cfb4dbc804b642e195340dbc9 0.3s done
#20 writing layer sha256:442077be437c11fc0fd11465f41b4cc8d779cedcb51d965fef673069fdc5b037
#20 writing layer sha256:442077be437c11fc0fd11465f41b4cc8d779cedcb51d965fef673069fdc5b037 0.3s done
#20 writing layer sha256:679c4fe0cc4400ea09dbd63fc8151ddaac9eaeacadac9dbbf03b906d93e5f892
#20 writing layer sha256:679c4fe0cc4400ea09dbd63fc8151ddaac9eaeacadac9dbbf03b906d93e5f892 0.3s done
#20 writing layer sha256:8890ddc07177137fe1cf3fa3a70910078e8980fe7ce5e1673c23a04b4f4a6971
#20 writing layer sha256:8890ddc07177137fe1cf3fa3a70910078e8980fe7ce5e1673c23a04b4f4a6971 0.3s done
#20 writing layer sha256:c7499e60179d65725a5d2d2838a08601b37ef43f9c6bfbeba08af04c11f3e755
#20 writing layer sha256:c7499e60179d65725a5d2d2838a08601b37ef43f9c6bfbeba08af04c11f3e755 0.3s done
#20 writing layer sha256:ced3cd4271313bdafd3b4feed113f4d467931d4e84ad73ca056315507f8165ad
#20 writing layer sha256:ced3cd4271313bdafd3b4feed113f4d467931d4e84ad73ca056315507f8165ad 0.3s done
#20 writing layer sha256:d9f4f37a2d6a074b249edd2089005c065f95f224c255f633a157a0fbbbac8fd1
#20 writing layer sha256:d9f4f37a2d6a074b249edd2089005c065f95f224c255f633a157a0fbbbac8fd1 0.3s done
#20 writing layer sha256:ddd404ee64e22560f285acd167f35bdc8fc1a7a966dad4d5693c29c4aed70d02
#20 writing layer sha256:ddd404ee64e22560f285acd167f35bdc8fc1a7a966dad4d5693c29c4aed70d02 0.3s done
#20 writing layer sha256:e3a15fb29df34b9eb719a1cd65bfa5f055410df9ec1b709d753bcbfbcc26bd09
#20 writing layer sha256:e3a15fb29df34b9eb719a1cd65bfa5f055410df9ec1b709d753bcbfbcc26bd09 0.3s done
#20 writing layer sha256:f82933bf1ac3376eba611609063f1515b8d9d03919bfab31bfe4df0536a73c6c
#20 writing layer sha256:f82933bf1ac3376eba611609063f1515b8d9d03919bfab31bfe4df0536a73c6c 0.3s done
#20 writing config sha256:f32e2823fba43d06109d9e13b0213dbf962443ee37b0287ab4386224e66a79f3
#20 writing config sha256:f32e2823fba43d06109d9e13b0213dbf962443ee37b0287ab4386224e66a79f3 0.3s done
#20 writing cache image manifest sha256:2565091c9179fd7faa94326e441b03929f706cf525c659cce494eb0acfa1ba7c
#20 preparing build cache for export 4.4s done
#20 writing cache image manifest sha256:2565091c9179fd7faa94326e441b03929f706cf525c659cce494eb0acfa1ba7c 0.2s done
#20 DONE 4.4s

</pre>
</details>

---

<details>
<summary>Pod buildkitd-arm</summary>
<br>
Logs.
<br><br>
<pre>

time="2025-06-24T12:42:36Z" level=warning msg="TLS is disabled for unix:///run/user/1000/buildkit/buildkitd.sock"
time="2025-06-24T12:42:36Z" level=info msg="auto snapshotter: using overlayfs"
time="2025-06-24T12:42:36Z" level=warning msg="NoProcessSandbox is enabled. Note that NoProcessSandbox allows build containers to kill (and potentially ptrace) an arbitrary process in the BuildKit host namespace. NoProcessSandbox should be enabled only when the BuildKit is running in a container as an unprivileged user."
time="2025-06-24T12:42:36Z" level=warning msg="CDI setup error /etc/cdi: failed to monitor for changes: no such file or directory"
time="2025-06-24T12:42:36Z" level=warning msg="CDI setup error /var/run/cdi: failed to monitor for changes: no such file or directory"
time="2025-06-24T12:42:36Z" level=warning msg="CDI setup error /etc/buildkit/cdi: failed to monitor for changes: no such file or directory"
time="2025-06-24T12:42:36Z" level=info msg="found worker \"dptgp7yorzm4afymsmmpieysg\", labels=map[org.mobyproject.buildkit.worker.executor:oci org.mobyproject.buildkit.worker.hostname:buildkitd-arm-7f6498ffb-tnt2p org.mobyproject.buildkit.worker.network:host org.mobyproject.buildkit.worker.oci.process-mode:no-sandbox org.mobyproject.buildkit.worker.selinux.enabled:false org.mobyproject.buildkit.worker.snapshotter:overlayfs], platforms=[linux/arm64 linux/amd64 linux/amd64/v2 linux/riscv64 linux/ppc64le linux/s390x linux/386 linux/arm/v7 linux/arm/v6]"
time="2025-06-24T12:42:36Z" level=warning msg="skipping containerd worker, as \"/run/containerd/containerd.sock\" does not exist"
time="2025-06-24T12:42:36Z" level=info msg="found 1 workers, default=\"dptgp7yorzm4afymsmmpieysg\""
time="2025-06-24T12:42:36Z" level=warning msg="currently, only the default worker can be used."
time="2025-06-24T12:42:36Z" level=info msg="running server on /run/user/1000/buildkit/buildkitd.sock"
time="2025-06-24T12:42:36Z" level=info msg="running server on [::]:1234"
time="2025-06-24T13:16:11Z" level=info msg="fetch failed after status: 404 Not Found" host=registry-1.docker.io
time="2025-06-24T13:16:18Z" level=warning msg="reference for unknown type: application/vnd.buildkit.cacheconfig.v0" spanID=f571e02d450f43f3 traceID=12ce6ee558044e79ad1e2ca62ef9ec14
time="2025-06-24T13:23:32Z" level=warning msg="reference for unknown type: application/vnd.buildkit.cacheconfig.v0" spanID=21634be1f5621be6 traceID=6b8e9b10763c69bf4225df5d39c2564c
time="2025-06-24T13:26:08Z" level=warning msg="reference for unknown type: application/vnd.buildkit.cacheconfig.v0" spanID=e77cec090168b164 traceID=b0b173f7c84d48bea1d9516fb9e87958
</pre>
</details>
---

![docker hub](/images/1.png)

![jager](/images/2.png)

![jager](/images/3.png)