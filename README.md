# CKAD — Study Plan & Progressive Mock Tests

**Format:** questions first (beginner → expert), **all corrections are at the end of this document**.
Do not scroll to the corrections until your timer is done.

---

## 0. Exam facts you need before planning

| Item | Value |
|---|---|
| Duration | 2 hours |
| Questions | 15–20 performance-based tasks, on live clusters |
| Passing score | 66% |
| Attempts | 1 free retake included |
| Environment | Remote desktop, browser terminal, multiple clusters |
| Allowed docs | `kubernetes.io/docs`, `kubernetes.io/blog`, `helm.sh/docs` (one extra browser tab) |
| Validity | 2 years |

### Curriculum weights

| Domain | Weight |
|---|---|
| Application Design and Build | 20% |
| Application Deployment | 20% |
| Application Observability and Maintenance | 15% |
| Application Environment, Configuration and Security | 25% |
| Services and Networking | 20% |

**The exam is a speed test, not a knowledge test.** Roughly 6–7 minutes per question. Anyone who writes YAML from scratch by hand fails on time.

---

## 1. Environment setup (do this on day 1, not on exam day)

```bash
# ~/.bashrc equivalent — set this up in every practice session
alias k=kubectl
export do="--dry-run=client -o yaml"     # k run nginx --image=nginx $do > pod.yaml
export now="--force --grace-period=0"    # k delete pod nginx $now
source <(kubectl completion bash)
complete -F __start_kubectl k
```

`~/.vimrc`:

```vim
set expandtab
set tabstop=2
set shiftwidth=2
set number
```

Practice cluster options: `kind`, `minikube`, or k3s on a VPS. `kind` with a 3-node config is closest to the exam.

```bash
cat <<EOF | kind create cluster --name ckad --config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
```

---

## 2. Six-week plan

Each week: 4 study days (~1h), 1 drill day (~45 min), 1 mock test day, 1 rest day.

### Week 1 — Core objects (Design & Build)
- Pods, multi-container patterns (sidecar, ambassador, adapter), init containers
- Labels, selectors, annotations
- Jobs, CronJobs (`completions`, `parallelism`, `backoffLimit`, `activeDeadlineSeconds`)
- Container images: writing a Dockerfile, `podman build`/`docker build`, tags
- **→ Mock Test 1 (Beginner)**

### Week 2 — Deployment & rollouts
- Deployments, ReplicaSets, rolling updates, `kubectl rollout undo/status/history`
- Deployment strategies: rolling, recreate, blue/green and canary *implemented with labels + services*
- Helm basics: `helm install/upgrade/rollback/list`, `values.yaml`, `helm template`
- Kustomize: `kustomization.yaml`, overlays, patches
- **→ Mock Test 2 (Intermediate)**

### Week 3 — Configuration & security
- ConfigMaps and Secrets: `env`, `envFrom`, volume mounts, `subPath`
- SecurityContext: `runAsUser`, `runAsNonRoot`, `fsGroup`, `capabilities`, `readOnlyRootFilesystem`
- ServiceAccounts, RBAC (Role/RoleBinding), `automountServiceAccountToken`
- Resource requests/limits, LimitRange, ResourceQuota
- **→ Drill: 20 config tasks in 40 minutes**

### Week 4 — Observability & maintenance
- Probes: liveness, readiness, startup (`exec`, `httpGet`, `tcpSocket`, timings)
- `kubectl logs` (`-f`, `--previous`, `-c`, `--since`), `kubectl describe`, `kubectl events`
- `kubectl debug`, ephemeral containers, `kubectl exec`
- API deprecations, `kubectl explain`, `kubectl api-resources`
- **→ Mock Test 3 (Advanced)**

### Week 5 — Services, networking, storage
- Services: ClusterIP, NodePort, LoadBalancer, headless
- Ingress, IngressClass, path types
- NetworkPolicy: ingress/egress, `podSelector`, `namespaceSelector`, `ipBlock`
- PV, PVC, StorageClass, `emptyDir`, `hostPath`
- **→ Drill: 15 networking tasks in 35 minutes**

### Week 6 — Simulation & speed
- Two full timed simulations under exam conditions (2h, one browser tab, no notes)
- Rework every mistake, then redo the same test in half the time
- **→ Mock Test 4 (Expert, full simulation)**

---

## 3. Mock Test 1 — Beginner (target: 35 minutes)

Namespace for all tasks: `ckad-b`. Create it first.

**B1.** Create a pod named `web` using image `nginx:1.25`, exposing container port `80`, with label `tier=frontend`. Use an imperative command, not a hand-written manifest.

**B2.** Create the same pod definition as a YAML file at `/tmp/web.yaml` *without* creating it in the cluster.

**B3.** Create a pod `busy` running image `busybox:1.36` that executes `sleep 3600`.

**B4.** List all pods in namespace `ckad-b` that carry the label `tier=frontend`, showing their labels.

**B5.** Add the label `env=dev` to the pod `web`, then annotate it with `owner=steve`.

**B6.** Create a Deployment named `api` with image `httpd:2.4`, 3 replicas, in namespace `ckad-b`.

**B7.** Scale the `api` deployment to 5 replicas, then expose it on port `80` as a ClusterIP service named `api-svc`.

**B8.** Create a Job named `pi` that runs `perl:5.34` with the command `perl -Mbignum=bpi -wle 'print bpi(2000)'`, completing 3 times, 2 at a time.

**B9.** Create a CronJob named `hello` that runs `busybox:1.36` every minute and echoes `Hello from CKAD`.

**B10.** Retrieve the logs of the pod `busy`, then delete it immediately with no grace period.

---

## 4. Mock Test 2 — Intermediate (target: 50 minutes)

Namespace: `ckad-i`.

**I1.** Create a pod `multi` with two containers: `app` (image `nginx:1.25`) and `log` (image `busybox:1.36`, command `sh -c 'tail -f /var/log/nginx/access.log'`). Both must share an `emptyDir` volume mounted at `/var/log/nginx`.

**I2.** Create a pod `initapp` (image `nginx:1.25`) with an init container `setup` (image `busybox:1.36`) that writes `ready` into `/work/status` on a shared `emptyDir` mounted at `/work` in both containers.

**I3.** Create a ConfigMap `app-config` with keys `APP_MODE=production` and `APP_TIER=backend`, then a pod `cfg` (image `nginx:1.25`) that injects **all** keys as environment variables.

**I4.** Create a Secret `db-cred` with `user=admin` and `password=S3cr3t`, then mount it as a volume at `/etc/db` in a pod named `secret-pod` (image `nginx:1.25`), read-only.

**I5.** Create a Deployment `blue` with image `nginx:1.24`, 3 replicas, label `app=web,version=blue`. Create a service `web-svc` selecting `app=web`. Then deploy `green` with `nginx:1.25` and switch traffic to green without downtime.

**I6.** Perform a rolling update of deployment `api` (from Test 1 style) to `httpd:2.4.58`, watch the rollout status, then roll it back and show the revision history.

**I7.** Create a pod `limited` (image `nginx:1.25`) with resource requests `cpu=100m, memory=128Mi` and limits `cpu=500m, memory=256Mi`.

**I8.** Create a pod `probes` (image `nginx:1.25`) with a readiness probe on HTTP `GET /` port 80 (initial delay 5s, period 5s) and a liveness probe running `cat /usr/share/nginx/html/index.html` every 10s with a failure threshold of 3.

**I9.** A ServiceAccount `deployer` must exist in `ckad-i`, and a pod `sa-pod` (image `nginx:1.25`) must run under it.

**I10.** Create a NodePort service exposing an existing deployment `api` on port `80`, target port `80`, node port `30080`.

---

## 5. Mock Test 3 — Advanced (target: 60 minutes)

Namespace: `ckad-a`.

**A1.** Deployment `secure-app` (image `nginx:1.25`, 2 replicas) must run as UID `1000`, GID `3000`, with `fsGroup=2000`, must not run as root, and must have a read-only root filesystem (mount an `emptyDir` at `/var/cache/nginx` and `/var/run` so nginx still starts).

**A2.** Create a Role `pod-reader` allowing `get`, `list`, `watch` on pods in `ckad-a`, and bind it to the ServiceAccount `deployer` in the same namespace. Verify with `kubectl auth can-i`.

**A3.** Apply a ResourceQuota `team-quota` to `ckad-a` limiting the namespace to 10 pods, 2 CPU requests and 4Gi memory limits. Then prove it works by trying to exceed it.

**A4.** Create a NetworkPolicy `db-policy` in `ckad-a` that allows ingress to pods labelled `role=db` on TCP `5432` **only** from pods labelled `role=api` in the same namespace, and denies everything else.

**A5.** Create a PersistentVolumeClaim `data-pvc` requesting 1Gi, `ReadWriteOnce`, storage class `standard`, and mount it at `/data` in a pod named `stateful-pod` (image `nginx:1.25`).

**A6.** Pod `broken` (image `nginx:1.25`) is in `CrashLoopBackOff`. Collect: the logs of its previous instance, the events for the pod, and the exit code of the last terminated container — using one command per item.

**A7.** A running pod `no-shell` uses a distroless image with no shell. Attach a debug container (image `busybox:1.36`) to inspect its filesystem without restarting it.

**A8.** Create a Deployment `canary-v2` so that roughly 20% of traffic through `web-svc` reaches version `v2`, using only labels, replicas and one service (no service mesh, no Ingress weights).

**A9.** Convert a pod manifest at `/tmp/legacy.yaml` that uses `apiVersion: extensions/v1beta1` for a Deployment into the current API version, and explain how you found it without the browser.

**A10.** Package a simple app with Helm: install chart `bitnami/nginx` as release `mysite` with `replicaCount=2`, then upgrade to `replicaCount=3` and roll back to revision 1.

---

## 6. Mock Test 4 — Expert / Full simulation (strict: 2 hours, 16 tasks)

Rules: one browser tab on the official docs only. No AI, no notes, no pausing. Score yourself out of 100; you need 66.

**E1** *(6 pts)* — In namespace `prod`, create a Deployment `frontend` (image `nginx:1.25`, 4 replicas) using a `RollingUpdate` strategy with `maxSurge=1` and `maxUnavailable=0`. Annotate the deployment with a change cause of `initial release`.

**E2** *(6 pts)* — Expose `frontend` through an Ingress `frontend-ing` on host `app.example.com`, path `/` with `pathType: Prefix`, routed to service `frontend-svc:80`. Create the service too.

**E3** *(7 pts)* — Create a CronJob `report` running every day at 03:30, image `busybox:1.36`, command `sh -c 'date; echo report done'`, keeping 3 successful and 1 failed job in history, killed if it runs longer than 30 seconds, and never running concurrently with itself.

**E4** *(7 pts)* — Create a pod `sidecar-logger` where the main container `app` writes a line to `/var/log/app.log` every second, and a sidecar `shipper` (busybox) tails that same file to stdout. Verify with `kubectl logs`.

**E5** *(6 pts)* — A Secret `api-key` holds key `token`. Mount **only that key** as a file at `/etc/secret/api-token` in pod `consumer` (image `nginx:1.25`), with file permission `0400`.

**E6** *(7 pts)* — Deployment `db` must only accept traffic from pods with label `app=api` in namespace `prod` and from anything in namespace `monitoring`. All egress from `db` must be denied except DNS (UDP/TCP 53).

**E7** *(6 pts)* — Pod `slow-start` (image `nginx:1.25`) takes up to 90 seconds to boot. Configure probes so Kubernetes does not kill it during startup but still restarts it if it hangs later.

**E8** *(6 pts)* — Find every pod in **all** namespaces that has no resource limits set, and output `namespace/name` only, one per line.

**E9** *(7 pts)* — Create a ServiceAccount `ci` in `prod` that can create and delete Deployments but only read Secrets. No token should be auto-mounted into pods using it.

**E10** *(6 pts)* — Deployment `web` is stuck: rollout never completes. Diagnose and fix. (Hint: inspect `kubectl rollout status`, events, and the pod template.)

**E11** *(6 pts)* — Using Kustomize, produce an overlay `prod` that takes a base deployment and changes the image tag to `1.25.3` and the replica count to 5. Show the rendered output without applying it.

**E12** *(6 pts)* — Create a pod `multi-cfg` that gets `LOG_LEVEL` from ConfigMap `app-cfg` key `log_level`, `DB_PASS` from Secret `db-cred` key `password`, and `POD_IP` from the downward API.

**E13** *(6 pts)* — Save the YAML of the running deployment `frontend` to `/tmp/frontend-backup.yaml`, cleaned of `status`, `creationTimestamp`, `resourceVersion` and `uid`, so it can be re-applied to another cluster.

**E14** *(6 pts)* — Create a headless service `db-headless` for deployment `db` on port `5432`, then resolve its pod DNS names from a temporary pod.

**E15** *(6 pts)* — Two containers in pod `shared` must communicate over `localhost:8080`, and a third container must be prevented from starting until a file `/data/ready` exists. Implement it.

**E16** *(6 pts)* — In namespace `prod`, one pod is consuming far more CPU than the others. Identify it and record its name in `/tmp/top-pod.txt`.

---

## 7. Speed drills (do these daily in week 6)

Time yourself. Target is in brackets.

1. Pod from scratch with env var + resource limits *(60s)*
2. Deployment + expose + scale *(75s)*
3. ConfigMap from literal + mount as volume *(90s)*
4. Secret from file + inject as env *(90s)*
5. NetworkPolicy from the docs, adapted *(150s)*
6. Ingress from the docs, adapted *(120s)*
7. Rollout, undo, verify revision *(60s)*
8. Find failing pod across all namespaces and read previous logs *(60s)*

**The docs pages worth bookmarking mentally** (you cannot save bookmarks, but you can learn the search terms): search `network policy` → copy the full example; search `ingress` → the minimal ingress; search `security context` → the pod example; search `persistent volume claim` → the PVC + pod example. Copy, then edit. Never type from memory.

---
---

# CORRECTIONS

> Everything below is the answer key. Compare command-by-command, not just outcome-by-outcome — on the exam, the slow-but-correct path still fails you.

---

## Corrections — Mock Test 1 (Beginner)

**Setup**

```bash
k create ns ckad-b
k config set-context --current --namespace=ckad-b
```

Always switch context. Typing `-n ckad-b` on 10 tasks costs you a minute.

**B1**

```bash
k run web --image=nginx:1.25 --port=80 --labels=tier=frontend
```

**B2**

```bash
k run web --image=nginx:1.25 --port=80 --labels=tier=frontend $do > /tmp/web.yaml
```

`$do` is `--dry-run=client -o yaml`. This is the single most valuable alias in the exam.

**B3**

```bash
k run busy --image=busybox:1.36 --command -- sleep 3600
```

The `--command` flag matters: without it, the arguments go to `args` instead of `command`, which for busybox happens to work, but for images with an `ENTRYPOINT` it silently does the wrong thing.

**B4**

```bash
k get pods -l tier=frontend --show-labels
```

**B5**

```bash
k label pod web env=dev
k annotate pod web owner=steve
```

To overwrite an existing label, add `--overwrite`.

**B6**

```bash
k create deploy api --image=httpd:2.4 --replicas=3
```

**B7**

```bash
k scale deploy api --replicas=5
k expose deploy api --name=api-svc --port=80 --target-port=80
```

`kubectl expose` reuses the deployment's selector automatically — never write the selector by hand.

**B8**

```bash
k create job pi --image=perl:5.34 $do -- perl -Mbignum=bpi -wle 'print bpi(2000)' > pi.yaml
```

Then edit to add:

```yaml
spec:
  completions: 3
  parallelism: 2
```

```bash
k apply -f pi.yaml
```

There is no imperative flag for `completions`/`parallelism` — dry-run then edit is the fastest route.

**B9**

```bash
k create cronjob hello --image=busybox:1.36 --schedule="* * * * *" -- /bin/sh -c 'echo Hello from CKAD'
```

**B10**

```bash
k logs busy
k delete pod busy --force --grace-period=0    # or: k delete pod busy $now
```

---

## Corrections — Mock Test 2 (Intermediate)

**I1** — Generate the base, then edit:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi
spec:
  volumes:
    - name: logs
      emptyDir: {}
  containers:
    - name: app
      image: nginx:1.25
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
    - name: log
      image: busybox:1.36
      command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
```

Common failure: forgetting that `tail -f` on a file that does not exist yet makes the sidecar crash-loop. `tail -F` or `sh -c 'touch ... && tail -f ...'` is safer in real life; on the exam, match the wording asked.

**I2**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: initapp
spec:
  volumes:
    - name: work
      emptyDir: {}
  initContainers:
    - name: setup
      image: busybox:1.36
      command: ["sh", "-c", "echo ready > /work/status"]
      volumeMounts:
        - name: work
          mountPath: /work
  containers:
    - name: nginx
      image: nginx:1.25
      volumeMounts:
        - name: work
          mountPath: /work
```

`initContainers` sits at the same level as `containers`, inside `spec`. Indentation mistakes here are the classic time sink.

**I3**

```bash
k create cm app-config --from-literal=APP_MODE=production --from-literal=APP_TIER=backend
k run cfg --image=nginx:1.25 $do > cfg.yaml
```

Add to the container:

```yaml
      envFrom:
        - configMapRef:
            name: app-config
```

`envFrom` injects all keys. Use `env` + `valueFrom.configMapKeyRef` only when a single key is asked for.

**I4**

```bash
k create secret generic db-cred --from-literal=user=admin --from-literal=password=S3cr3t
```

```yaml
spec:
  volumes:
    - name: cred
      secret:
        secretName: db-cred
  containers:
    - name: secret-pod
      image: nginx:1.25
      volumeMounts:
        - name: cred
          mountPath: /etc/db
          readOnly: true
```

**I5** — Blue/green with labels:

```bash
k create deploy blue --image=nginx:1.24 --replicas=3
k label deploy blue app=web version=blue --overwrite
# ensure pod template labels carry version=blue:
k set selector ...   # not needed if you generate the manifest instead
```

The reliable way is to write both deployments with explicit pod-template labels `app=web` plus `version=blue|green`, create the service selecting only `app=web,version=blue`, then switch:

```bash
k patch svc web-svc -p '{"spec":{"selector":{"app":"web","version":"green"}}}'
```

The switch is atomic and instant. That is the whole point of blue/green.

**I6**

```bash
k set image deploy/api httpd=httpd:2.4.58
k rollout status deploy/api
k rollout undo deploy/api
k rollout history deploy/api
```

Note `httpd=` is the **container** name, not the deployment name. Check it with `k get deploy api -o jsonpath='{.spec.template.spec.containers[*].name}'`.

**I7**

```yaml
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 256Mi
```

Imperative shortcut: `k run limited --image=nginx:1.25 --requests=cpu=100m,memory=128Mi --limits=cpu=500m,memory=256Mi` (deprecated in recent versions — verify on your cluster version; if it fails, dry-run and edit).

**I8**

```yaml
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    livenessProbe:
      exec:
        command: ["cat", "/usr/share/nginx/html/index.html"]
      periodSeconds: 10
      failureThreshold: 3
```

**I9**

```bash
k create sa deployer
k run sa-pod --image=nginx:1.25 $do > sa-pod.yaml
```

Add `serviceAccountName: deployer` under `spec:` (not under the container).

**I10**

```bash
k expose deploy api --name=api-np --type=NodePort --port=80 --target-port=80
k patch svc api-np -p '{"spec":{"ports":[{"port":80,"targetPort":80,"nodePort":30080}]}}'
```

`kubectl expose` cannot set a specific `nodePort`; patch or edit afterwards.

---

## Corrections — Mock Test 3 (Advanced)

**A1**

```yaml
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    runAsNonRoot: true
  volumes:
    - name: cache
      emptyDir: {}
    - name: run
      emptyDir: {}
  containers:
    - name: nginx
      image: nginx:1.25
      securityContext:
        readOnlyRootFilesystem: true
        allowPrivilegeEscalation: false
      volumeMounts:
        - name: cache
          mountPath: /var/cache/nginx
        - name: run
          mountPath: /var/run
```

Pod-level `securityContext` holds `runAsUser`, `fsGroup`, `runAsNonRoot`. Container-level holds `readOnlyRootFilesystem`, `capabilities`, `allowPrivilegeEscalation`. Mixing the two up is the most common mistake in this domain.

**A2**

```bash
k create role pod-reader --verb=get,list,watch --resource=pods
k create rolebinding pod-reader-rb --role=pod-reader --serviceaccount=ckad-a:deployer
k auth can-i list pods --as=system:serviceaccount:ckad-a:deployer -n ckad-a   # -> yes
```

For a ServiceAccount, the flag is `--serviceaccount=NAMESPACE:NAME`. For a user it is `--user=`.

**A3**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: ckad-a
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    limits.memory: 4Gi
```

Once a quota specifying `requests.cpu`/`limits.memory` exists, **every new pod must declare those values** or creation is rejected. That rejection is the proof asked for:

```bash
k run q --image=nginx   # -> error: must specify limits.memory
```

**A4**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
  namespace: ckad-a
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: api
      ports:
        - protocol: TCP
          port: 5432
```

Because a policy selects `role=db`, everything else is denied to those pods implicitly. Watch the YAML list structure: `from:` and `ports:` are siblings inside a single ingress rule. Putting `ports` at the wrong indent creates an "allow from anywhere on 5432" rule — a silent, scored failure.

**A5**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: stateful-pod
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: data-pvc
  containers:
    - name: nginx
      image: nginx:1.25
      volumeMounts:
        - name: data
          mountPath: /data
```

**A6**

```bash
k logs broken --previous
k describe pod broken            # events at the bottom
k get pod broken -o jsonpath='{.status.containerStatuses[0].lastState.terminated.exitCode}'
```

`--previous` (or `-p`) is what gets you the crashed instance's output. Exit code 137 = OOMKilled, 1 = application error, 143 = SIGTERM.

**A7**

```bash
k debug -it no-shell --image=busybox:1.36 --target=no-shell
```

`--target` shares the process namespace of the named container. Without it you get a separate container that cannot see the target's processes. `kubectl debug` never restarts the pod — that is the point of ephemeral containers.

**A8** — Canary with replica ratio:

```bash
k create deploy canary-v2 --image=nginx:1.25 --replicas=1
k label deploy canary-v2 app=web --overwrite
```

The service `web-svc` selects `app=web` only. With 4 stable replicas and 1 canary replica, ~20% of requests land on v2. Both deployments' **pod templates** must carry `app=web`; a label on the Deployment object itself does nothing for service routing. This ratio-based split is the only canary CKAD expects.

**A9**

```bash
k explain deployment                 # shows the current apiVersion
k api-resources | grep -i deployment # -> apps/v1
```

Then edit `/tmp/legacy.yaml` to `apiVersion: apps/v1` and add the now-mandatory `spec.selector.matchLabels` matching the pod template labels. `kubectl explain --recursive deployment.spec` is your offline documentation for any field you forget.

**A10**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mysite bitnami/nginx --set replicaCount=2
helm upgrade mysite bitnami/nginx --set replicaCount=3
helm history mysite
helm rollback mysite 1
```

`helm template` renders without installing — useful when a task asks what a chart *would* create.

---

## Corrections — Mock Test 4 (Expert)

**E1**

```bash
k create deploy frontend -n prod --image=nginx:1.25 --replicas=4 $do > fe.yaml
```

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

```bash
k apply -f fe.yaml
k annotate deploy frontend -n prod kubernetes.io/change-cause="initial release"
```

`maxUnavailable: 0` guarantees zero downtime but requires cluster capacity for one extra pod.

**E2**

```bash
k expose deploy frontend -n prod --name=frontend-svc --port=80 --target-port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ing
  namespace: prod
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-svc
                port:
                  number: 80
```

**E3**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: report
spec:
  schedule: "30 3 * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      activeDeadlineSeconds: 30
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: report
              image: busybox:1.36
              command: ["sh", "-c", "date; echo report done"]
```

`activeDeadlineSeconds` goes on the **Job** spec (kills the run), not on the CronJob spec (`startingDeadlineSeconds` there means something else entirely — how late a missed schedule may still start).

**E4**

```yaml
spec:
  volumes:
    - name: logs
      emptyDir: {}
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "while true; do echo $(date) >> /var/log/app.log; sleep 1; done"]
      volumeMounts:
        - name: logs
          mountPath: /var/log
    - name: shipper
      image: busybox:1.36
      command: ["sh", "-c", "tail -F /var/log/app.log"]
      volumeMounts:
        - name: logs
          mountPath: /var/log
```

```bash
k logs sidecar-logger -c shipper
```

**E5**

```yaml
  volumes:
    - name: sec
      secret:
        secretName: api-key
        defaultMode: 0400
        items:
          - key: token
            path: api-token
  containers:
    - name: nginx
      image: nginx:1.25
      volumeMounts:
        - name: sec
          mountPath: /etc/secret
```

`items` restricts which keys are projected and renames the file. Without it, every key in the secret appears.

**E6**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes: [Ingress, Egress]
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: monitoring
  egress:
    - ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

Two entries under one `from:` list = OR. Combining `podSelector` and `namespaceSelector` inside a **single** list item = AND. That distinction — one dash or two — is the single highest-value detail in the networking domain.

**E7**

```yaml
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 5          # 30 x 5 = 150s budget
    livenessProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 10
      failureThreshold: 3
```

The liveness probe is disabled until the startup probe succeeds. Using a large `initialDelaySeconds` on liveness instead is the wrong answer: it delays detection forever, not just during boot.

**E8**

```bash
k get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}{"\t"}{.spec.containers[*].resources.limits}{"\n"}{end}' \
  | grep -vP '\t.+' | cut -f1
```

Simpler and accepted if the task only asks you to list them:

```bash
k get pods -A -o json | jq -r '.items[] | select(any(.spec.containers[]; .resources.limits == null)) | "\(.metadata.namespace)/\(.metadata.name)"'
```

`jq` is available in the exam environment. Learn the `-o jsonpath={range}` form anyway — it is the guaranteed-present tool.

**E9**

```bash
k create sa ci -n prod
k create role ci-role -n prod --verb=create,delete,get,list --resource=deployments
k create role ci-secrets -n prod --verb=get,list --resource=secrets
k create rolebinding ci-rb -n prod --role=ci-role --serviceaccount=prod:ci
k create rolebinding ci-secrets-rb -n prod --role=ci-secrets --serviceaccount=prod:ci
```

Then disable token mounting on the ServiceAccount:

```bash
k patch sa ci -n prod -p '{"automountServiceAccountToken": false}'
```

`automountServiceAccountToken` can be set on the ServiceAccount (applies to all pods using it) or on the pod spec (overrides the SA). The task says "pods using it" → set it on the SA.

**E10** — Diagnostic order, always the same:

```bash
k rollout status deploy/web --timeout=10s
k get pods -l app=web
k describe pod <one-pending-or-crashing-pod>   # read Events
k get events --sort-by=.lastTimestamp
```

The three realistic causes: an image that does not exist (`ErrImagePull` / `ImagePullBackOff`), a resource request no node can satisfy (`Pending`, `FailedScheduling: Insufficient cpu`), or a readiness probe that never passes (pods `Running` but `0/1 READY`, rollout stalls forever with `maxUnavailable: 0`). Fix by `k set image`, by lowering requests, or by correcting the probe path/port.

**E11**

```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
images:
  - name: nginx
    newTag: 1.25.3
replicas:
  - name: web
    count: 5
```

```bash
k kustomize overlays/prod        # render only
k apply -k overlays/prod         # render and apply
```

**E12**

```yaml
      env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-cfg
              key: log_level
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: db-cred
              key: password
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
```

Downward API fields worth memorising: `metadata.name`, `metadata.namespace`, `status.podIP`, `spec.nodeName`, and for resources `resourceFieldRef`.

**E13**

```bash
k get deploy frontend -n prod -o yaml > /tmp/frontend-backup.yaml
```

Then strip the runtime fields. The fast, reliable way:

```bash
k get deploy frontend -n prod -o yaml \
  | k neat > /tmp/frontend-backup.yaml     # if the kubectl-neat plugin exists
```

Assume it does not. Delete by hand in vim: the whole `status:` block, plus `metadata.creationTimestamp`, `metadata.resourceVersion`, `metadata.uid`, `metadata.generation`, `metadata.selfLink`, and the `kubectl.kubernetes.io/last-applied-configuration` annotation. In vim, `/status:` then `dG` removes everything from `status:` to end of file — that alone handles most of it.

**E14**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None
  selector:
    app: db
  ports:
    - port: 5432
      targetPort: 5432
```

```bash
k run tmp --rm -it --image=busybox:1.36 --restart=Never -- nslookup db-headless.prod.svc.cluster.local
```

`clusterIP: None` makes DNS return the pod IPs directly instead of a single virtual IP. `--rm -it --restart=Never` is the throwaway-pod pattern you should be able to type without thinking.

**E15**

Containers in the same pod already share a network namespace, so `localhost:8080` works with no configuration — just have one container listen on 8080. The gating requirement is the init container:

```yaml
spec:
  volumes:
    - name: data
      emptyDir: {}
  initContainers:
    - name: wait-ready
      image: busybox:1.36
      command: ["sh", "-c", "until [ -f /data/ready ]; do sleep 2; done"]
      volumeMounts:
        - name: data
          mountPath: /data
  containers:
    - name: server
      image: nginx:1.25
    - name: client
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
```

A trap in this question: init containers run **before** all app containers, so if the `/data/ready` file is meant to be produced by another app container in the same pod, the pod deadlocks. If the file comes from outside (a volume, an operator), it works.

**E16**

```bash
k top pods -n prod --sort-by=cpu
k top pods -n prod --sort-by=cpu --no-headers | head -1 | awk '{print $1}' > /tmp/top-pod.txt
```

`k top` needs metrics-server. If it errors, that is the answer to a different question — but on the exam it is installed.

---

## Scoring and what to do next

| Score on Mock 4 | Read this |
|---|---|
| < 50% | Do not book yet. Redo weeks 1–3, focus on generating manifests imperatively. |
| 50–65% | You know the material; you are losing time. Drill section 7 daily for a week. |
| 66–80% | Book the exam. Redo Mock 3 and 4 with a 90-minute limit. |
| > 80% | Book it. Spend remaining time on NetworkPolicy and securityContext only — they carry the most partial-credit loss. |

**Exam-day rules that are worth points:**

1. Set the namespace context for every question. Re-read the namespace line twice.
2. Flag and skip anything that costs more than 8 minutes. Come back at the end.
3. Copy from the docs, then edit. Never author YAML from memory.
4. Verify each answer (`k get`, `k describe`, `k logs`) before moving on — partial credit exists, but a resource that failed to create scores zero.
5. Leave 10 minutes at the end for the flagged questions and a final `k get all -A` sanity pass.
