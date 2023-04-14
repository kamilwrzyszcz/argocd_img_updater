## Argo CD example with image updater and private repo
```
minikube start
terraform apply
kubectl port-forward svc/argocd-server -n argocd 8080:80
kubectl get secrets argocd-initial-admin-secret -o yaml -n argocd
echo "secret_here" | base64 -d
```
  
Public key as a deployment key in private repo, private key as a sealed secret:  
`ssh-keygen -t ed25519 -C "your@email.com" -f ~/.ssh/argocd_ed25519`  
  
Initial push of the image:  
```
docker tag nginx kwrzyszcz/nginx_test:v0.1.0
docker push kwrzyszcz/nginx_test:v0.1.0
```
  
Sealing secret:  
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo sealed
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.20.2/kubeseal-0.20.2-linux-amd64.tar.gz
tar -xvzf kubeseal-0.20.2-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
kubeseal --controller-name sealed-secrets -o yaml -n kube-system <manifests/original-secret.yaml > manifests/sealed-secret.yaml
```
  
Deploy secret and argo:  
```
kubectl apply -f manifests/sealed-secret.yaml
kubectl apply -f manifests/application.yaml
```

Simulate CI pipeline and trigger argo to commit tag into private repo and redeploy the app:  
```
docker tag nginx kwrzyszcz/nginx_test:v0.1.1
docker push kwrzyszcz/nginx_test:v0.1.1
```