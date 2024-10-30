# kubernetes-test
Kubernetes Test Documentation

## Wymagane skrypty
1. `setup-cluster.sh` - Konfiguracja klastra
2. `deploy-app.sh` - Wdrożenie aplikacji
3. `test-app.sh` - Testy automatyczne
4. `deploy-git.sh` - Wdrożenie z repozytorium

## Scenariusze testowe

+ [DevOpsTerminal/kubernetes: scripts](https://github.com/DevOpsTerminal/kubernetes/tree/main)

### 1. Test konfiguracji klastra
```bash
./setup-cluster.sh <MASTER_IP> <WORKER_IP>
```
Walidacja:
- Status node'ów: `kubectl get nodes`
- Komponenty systemowe: `kubectl get pods -n kube-system`
- Networking CNI: `kubectl get pods -n kube-flannel`

Kryteria akceptacji:
- [ ] Wszystkie node'y w stanie Ready
- [ ] CoreDNS działa
- [ ] Flannel CNI aktywny

### 2. Test aplikacji
```bash
./deploy-app.sh <MASTER_IP>
```
Walidacja:
- Deployment: `kubectl get deployments`
- Pody: `kubectl get pods`
- Serwis: `kubectl get svc`
- Ingress: `kubectl get ingress`

Kryteria akceptacji:
- [ ] Deployment ma 2 repliki
- [ ] Wszystkie pody Running
- [ ] LoadBalancer ma zewnętrzne IP
- [ ] HTTPS działa

### 3. Test wdrożenia z Git
```bash
./deploy-git.sh <MASTER_IP> <GIT_URL>
```
Walidacja:
- Pliki konfiguracyjne: `ls -la k8s-temp/*.yaml`
- Status wdrożenia: `kubectl get all -n example`

Kryteria akceptacji:
- [ ] Repo zostało sklonowane
- [ ] Wszystkie pliki YAML zaaplikowane
- [ ] Zasoby działają w namespace example

## Diagnostyka błędów

### Network Plugin Not Ready
```bash
# Sprawdź CNI
ls /etc/cni/net.d/
kubectl logs -n kube-system -l k8s-app=flannel
```

### Pody w stanie Pending
```bash
# Sprawdź tainty
kubectl describe nodes | grep Taint
kubectl describe pod <POD_NAME>
```

### LoadBalancer Pending
```bash
# Sprawdź serwis
kubectl describe service <SERVICE_NAME>
# Sprawdź eventy
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Logi systemowe
```bash
# Kubelet
journalctl -u kubelet

# Containerd
journalctl -u containerd

# API Server
kubectl logs -n kube-system kube-apiserver-*
```

## Monitorowanie

### Zasoby
```bash
kubectl top nodes
kubectl top pods
```

### Stan klastra
```bash
kubectl get componentstatuses
kubectl get events --all-namespaces
```

### Networking
```bash
kubectl get endpoints
netstat -tulpn
```