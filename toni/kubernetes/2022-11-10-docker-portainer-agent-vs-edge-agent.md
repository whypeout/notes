# Menambahkan Instance ke Portainer Server sebagai Agent atau Edge ?

Edge Agent dibuat utk memanage Edge Compute env dimana device tidak bisa membuka port
seperti pada Portainer Agent biasa.

## Prep

### Expose Tunnel Port

Edge Agent berkomunikasi dg Portainer Server via Port 8000 (atau 30776 pada Kubernetes NodePort).
lewat port ini, Edge Agent bisa menarik Portainer instance, connect ke Portainer, melihat jika perlu,
kemunian memulai tunnel/menerima config update. Tanpa tunnel port terbuka disisi Portainer Server, kita tidak
bisa mengakses Edge endpoint. Jika kita sudah men-deploy Portainer tanpa membuka port ini,
kita perlu mendeploy ulang Portainer dg membuka port ini. opsi CLI `--tunnel-port` utk mengatur port berbeda
jika port tersebut sudah dipakai.

### Metode Deployment

- Portainer dg TLS : Jika Portainer menggunakan TLS, Agent akan melakukan koneksi via HTTPS ke Portainer (Recommended).
- Portainer dg Self-Signed Certs : Jika Portainer menggunakan Self Signed Certificates, Edge Agent harus di deploy dg opsi `-e EDGE_INSECURE_POLL=1`.
  tanpa flag ini, edge tidak bisa komunikasi dg Portainer.
- Portainer Fallback to HTTP : jika portainer tidak menggunakan konfig diatas, akan menggunakan HTTP utk agent polling sbg fallback.
  Not Recommended karena tidak secure.
  
## Menambahkan Edge Env ke Portainer

Buka Portainer, Masuk Menu **Environment**, kemudian **Add Environment**

![add-env](https://2914113074-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FUeb1lPrWdn7TlqnXoAAC%2Fuploads%2FCN9PzdDVV5WbVUfQ35Hi%2F2.16-environments-add.gif?alt=media&token=4b9a1b8a-38a1-4e97-8b83-f3d7e906f450)

Pilih Docker / Kubernetes tergantung Environment yg digunakan. klik **Start Wizard**.
kemudian pilih **Edge Agent**. masukkan details env sbb:
- Name : Nama Env, Misal KubeLokal
- Portainer Server URL : URL dan Port dari Portainer Server sbgmn terlihat di Edge Env. misal `https://192.168.200.123:9443` atau `https://kindprot.magggis.my.id`
  gunakan FQDN jika DNS (lokal) tersedia.
![fqdn](https://2914113074-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FUeb1lPrWdn7TlqnXoAAC%2Fuploads%2Fu263JgzO29fvsS7nWLZ9%2F2.15-settings-env-addenv-edge-name.png?alt=media&token=05bdd712-44b8-44cb-b86b-c86047349872)

sbg Opsional Step, expand **More Settings** dan sesuaikan Poll freq - mengatur berapa sering Edge Agent
akan mengecek Portainer Server utk new jobs. default every 5 seconds.
kita bisa mengkategorikan env dg menambahkan **group** atau **tag**
klik **Create** setelah selesai konfig

![01-kubernetes-edge-agent](https://user-images.githubusercontent.com/89820226/201574838-fc3b9302-9cb8-4dad-9889-e67b22b2519d.png)

![02-kubernetes-created](https://user-images.githubusercontent.com/89820226/201574865-03992ea6-2f2a-4677-adfe-b02358f8cc46.png)

ceklis `Allow self-signed certs`, secara default portainer menggunakan self-signed cert, atau kita bisa gunakan cert kita (letsencrypt) setelah diinstall

contoh: portainer server url: https://192.168.200.123:9443 dan tcp://192.168.200.123:30776, or https://kindprot.magggis.my.id dan tcp://kindprot.magggis.my.id:30776

- portainer edge agent for kubernetes env

```
curl https://downloads.portainer.io/ee2-16/portainer-edge-agent-setup.sh | bash -s -- "a92bc2b9-8bda-415e-af5e-b29950db27e2" "aHR0cHM6Ly8xOTIuMTY4LjIwMC4xMjM6OTQ0M3wxOTIuMTY4LjIwMC4xMjM6MzA3NzZ8NGI6MWM6Y2I6NmQ6NTQ6MjE6ZDE6MDg6YmY6MWE6MGQ6ZTM6MzQ6ZTY6YWI6N2Z8Mw" "1" "" ""
```

- [https://downloads.portainer.io/ee2-16/portainer-edge-agent-setup.sh](https://downloads.portainer.io/ee2-16/portainer-edge-agent-setup.sh)

```sh
#!/usr/bin/env bash

# Script used to deploy the Portainer Edge agent inside a Kubernetes cluster.

# Requires:
# curl
# kubectl

### COLOR OUTPUT ###

ESeq="\x1b["
RCol="$ESeq"'0m'    # Text Reset

# Regular               Bold                    Underline               High Intensity          BoldHigh Intens         Background              High Intensity Backgrounds
Bla="$ESeq"'0;30m';     BBla="$ESeq"'1;30m';    UBla="$ESeq"'4;30m';    IBla="$ESeq"'0;90m';    BIBla="$ESeq"'1;90m';   On_Bla="$ESeq"'40m';    On_IBla="$ESeq"'0;100m';
Red="$ESeq"'0;31m';     BRed="$ESeq"'1;31m';    URed="$ESeq"'4;31m';    IRed="$ESeq"'0;91m';    BIRed="$ESeq"'1;91m';   On_Red="$ESeq"'41m';    On_IRed="$ESeq"'0;101m';
Gre="$ESeq"'0;32m';     BGre="$ESeq"'1;32m';    UGre="$ESeq"'4;32m';    IGre="$ESeq"'0;92m';    BIGre="$ESeq"'1;92m';   On_Gre="$ESeq"'42m';    On_IGre="$ESeq"'0;102m';
Yel="$ESeq"'0;33m';     BYel="$ESeq"'1;33m';    UYel="$ESeq"'4;33m';    IYel="$ESeq"'0;93m';    BIYel="$ESeq"'1;93m';   On_Yel="$ESeq"'43m';    On_IYel="$ESeq"'0;103m';
Blu="$ESeq"'0;34m';     BBlu="$ESeq"'1;34m';    UBlu="$ESeq"'4;34m';    IBlu="$ESeq"'0;94m';    BIBlu="$ESeq"'1;94m';   On_Blu="$ESeq"'44m';    On_IBlu="$ESeq"'0;104m';
Pur="$ESeq"'0;35m';     BPur="$ESeq"'1;35m';    UPur="$ESeq"'4;35m';    IPur="$ESeq"'0;95m';    BIPur="$ESeq"'1;95m';   On_Pur="$ESeq"'45m';    On_IPur="$ESeq"'0;105m';
Cya="$ESeq"'0;36m';     BCya="$ESeq"'1;36m';    UCya="$ESeq"'4;36m';    ICya="$ESeq"'0;96m';    BICya="$ESeq"'1;96m';   On_Cya="$ESeq"'46m';    On_ICya="$ESeq"'0;106m';
Whi="$ESeq"'0;37m';     BWhi="$ESeq"'1;37m';    UWhi="$ESeq"'4;37m';    IWhi="$ESeq"'0;97m';    BIWhi="$ESeq"'1;97m';   On_Whi="$ESeq"'47m';    On_IWhi="$ESeq"'0;107m';

printSection() {
    echo -e "${BIYel}>>>> ${BIWhi}${1}${RCol}"
}

info() {
    echo -e "${BIWhi}${1}${RCol}"
}

success() {
    echo -e "${BIGre}${1}${RCol}"
}

error() {
    echo -e "${BIRed}${1}${RCol}"
}

errorAndExit() {
    echo -e "${BIRed}${1}${RCol}"
    exit 1
}

### !COLOR OUTPUT ###

parseEnvVars() {
  envs=""
  local ENV_SOURCE=$1
  IFS="," read -r -a env_array <<< "$ENV_SOURCE"
  for env in "${env_array[@]}"
  do
    IFS="=" read -r -a env_pair <<< "$env"
    local key="${env_pair[0]}"
    local value="${env_pair[1]:-$(eval "echo \$$key")}"
    envs="$envs --from-literal=$key=$value"
  done
  echo "$envs"
}

main() {
    if [[ $# -lt 2 ]]; then
        error "Not enough arguments"
        error "Usage: ${0} <EDGE_ID> <EDGE_KEY> <EDGE_INSECURE_POLL> <EDGE_SECRET:optional>"
        exit 1
    fi
    
    local EDGE_ID="$1"
    local EDGE_KEY="$2"
    local EDGE_INSECURE_POLL="$3"
    local EDGE_SECRET="$4"
    local ENV_SOURCE="$5"
    
    [[ "$(command -v curl)" ]] || errorAndExit "Unable to find curl binary. Please ensure curl is installed before running this script."
    [[ "$(command -v kubectl)" ]] || errorAndExit "Unable to find kubectl binary. Please ensure kubectl is installed before running this script."
    
    info "Downloading agent manifest..."
    curl -L  https://downloads.portainer.io/ee2-16/portainer-agent-edge-k8s.yaml -o portainer-agent-edge-k8s.yaml || errorAndExit "Unable to download agent manifest"
    
    info "Creating Portainer namespace..."
    kubectl create namespace portainer
    
    info "Creating agent configuration..."
    configmapCmd="kubectl create configmap -n portainer portainer-agent-edge"
    configmapCmd+=" --from-literal="EDGE_ID=$EDGE_ID""
    configmapCmd+=" --from-literal="EDGE_INSECURE_POLL=$EDGE_INSECURE_POLL""
    if [ -n "$EDGE_SECRET" ]; then
        configmapCmd+=" --from-literal="EDGE_SECRET=$EDGE_SECRET""
    fi
    
    if [[ -n "$ENV_SOURCE" ]]; then
        configmapCmd="$configmapCmd $(parseEnvVars "$ENV_SOURCE")"
    fi
    echo "$configmapCmd"
    $configmapCmd

    info "Creating agent secret..."
    kubectl create secret generic portainer-agent-edge-key "--from-literal=edge.key=$EDGE_KEY" -n portainer
    
    info "Deploying agent..."
    kubectl apply -f portainer-agent-edge-k8s.yaml || errorAndExit "Unable to deploy agent manifest"
    
    success "Portainer Edge agent successfully deployed"
    exit 0
}

main "$@"
```

---

# Portainer server instance

Deploy Portainer server instance

```
docker run -d \
--name=portainer \
--restart=on-failure \
-p 9000:9000 \
-p 8000:8000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
portainer/portainer-ce:latest
```

Portainer Edge Agent for Docker Standalone

- portainer edge agent for docker standalone

```
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  -v /:/host \
  -v portainer_agent_data:/data \
  --restart always \
  -e EDGE=1 \
  -e EDGE_ID=a92bc2b9-8bda-415e-af5e-b29950db27e2 \
  -e EDGE_KEY=aHR0cHM6Ly8xOTIuMTY4LjIwMC4xMjM6OTQ0M3wxOTIuMTY4LjIwMC4xMjM6MzA3NzZ8NGI6MWM6Y2I6NmQ6NTQ6MjE6ZDE6MDg6YmY6MWE6MGQ6ZTM6MzQ6ZTY6YWI6N2Z8Mw \
  -e EDGE_INSECURE_POLL=1 \
  --name portainer_edge_agent \
  portainer/agent:2.16.0
```

- portainer edge agent for docker standalone on windows

```
docker run -d \
  --mount type=npipe,src=\\.\pipe\docker_engine,dst=\\.\pipe\docker_engine \
  --mount type=bind,src=C:\ProgramData\docker\volumes,dst=C:\ProgramData\docker\volumes \
  --mount type=volume,src=portainer_agent_data,dst=C:\data \
  --restart always \
   -e EDGE=1 \
  -e EDGE_ID=a92bc2b9-8bda-415e-af5e-b29950db27e2 \
  -e EDGE_KEY=aHR0cHM6Ly8xOTIuMTY4LjIwMC4xMjM6OTQ0M3wxOTIuMTY4LjIwMC4xMjM6MzA3NzZ8NGI6MWM6Y2I6NmQ6NTQ6MjE6ZDE6MDg6YmY6MWE6MGQ6ZTM6MzQ6ZTY6YWI6N2Z8Mw \
  -e EDGE_INSECURE_POLL=1 \
  --name portainer_edge_agent \
  portainer/agent:2.16.0
```

### Portainer Edge Agent Upgrade via YAML Manifests

```
curl -L https://downloads.portainer.io/ce2-16/portainer-agent-edge-k8s.yaml -o portainer-agent-edge-k8s.yaml  

kubectl apply -f portainer-agent-edge-k8s.yaml
```

> saat upgrade, pastikan utk download YAML ter-update, kemudian apply ke existing env.

- [https://downloads.portainer.io/ce2-16/portainer-agent-edge-k8s.yaml](https://downloads.portainer.io/ce2-16/portainer-agent-edge-k8s.yaml)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: portainer
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: portainer-sa-clusteradmin
  namespace: portainer
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: portainer-crb-clusteradmin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: portainer-sa-clusteradmin
    namespace: portainer
# Optional: can be added to expose the agent port 80 to associate an Edge key.
# ---
# apiVersion: v1
# kind: Service
# metadata:
#   name: portainer-agent
#   namespace: portainer
# spec:
#   type: LoadBalancer
#   selector:
#     app: portainer-agent
#   ports:
#     - name: http
#       protocol: TCP
#       port: 80
#       targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: portainer-agent
  namespace: portainer
spec:
  clusterIP: None
  selector:
    app: portainer-agent
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: portainer-agent
  namespace: portainer
spec:
  selector:
    matchLabels:
      app: portainer-agent
  template:
    metadata:
      labels:
        app: portainer-agent
    spec:
      serviceAccountName: portainer-sa-clusteradmin
      containers:
        - name: portainer-agent
          image: portainer/agent:2.16.1
          imagePullPolicy: Always
          env:
            - name: LOG_LEVEL
              value: INFO
            - name: KUBERNETES_POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: EDGE
              value: "1"
            - name: AGENT_CLUSTER_ADDR
              value: "portainer-agent"
            - name: AGENT_SECRET
              valueFrom:
                configMapKeyRef:
                  name: portainer-agent-edge
                  key: EDGE_SECRET 
                  optional: true
            - name: EDGE_KEY
              valueFrom:
                secretKeyRef:
                  name: portainer-agent-edge-key
                  key: edge.key
          envFrom:
            - configMapRef:
                name: portainer-agent-edge
          ports:
            - containerPort: 9001
              protocol: TCP
            - containerPort: 80
              protocol: TCP
```

