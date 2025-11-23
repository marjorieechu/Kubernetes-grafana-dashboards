kind get clusters
kubectl config get-contexts  # Check cluster already in Kubeconfig
kubectl config current-context
kubectl config use-context kind-marjcluster
kubectl create namespace monitor   #creat namespace monitor
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts    # Not namespace specific
helm repo update   # Not namespace specific either. clusterwide
helm install kps prometheus-community/kube-prometheus-stack -n monitor --create-namespace   #A namespace need to be specified

kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitor get pods -l "release=kps"

Get Grafana 'admin' user password by running:
  kubectl --namespace monitor get secrets kps-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:
  export POD_NAME=$(kubectl --namespace monitor get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=kps" -oname)
  kubectl --namespace monitor port-forward $POD_NAME 3000

Get your grafana admin user password by running:
  kubectl get secret --namespace monitor -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo  



kubectl get svc -n monitor
identify grafana service and patch to NodePort
kubectl patch svc kps-grafana -n monitor -p '{"spec": {"type": "NodePort"}}'
kubectl port-forward svc/kps-grafana -n monitor 3000:80
127.0.0.1:3000 ; copy and paste on browser to confirm
Leave current shell to maintain the port forwarding.
open another shell to echo password with this command 
kubectl --namespace monitor get secrets kps-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo


Once logged in:Navigate to Dashboards → Manage → Folder “Kubernetes / Cluster” etc.
You will see dashboards like:
Kubernetes / Compute Resources / Cluster
Kubernetes / Compute Resources / Node
Kubernetes / Pods / Overview
Node Exporter Full etc.
These dashboards are automatically provisioned by kube-prometheus-stack.

Export dashboard JSON
There are two ways to get the JSON:
Option A — Export from Grafana UI
Open a dashboard in Grafana.
Click the gear icon (⚙️) → JSON Model.
Click Copy JSON to Clipboard or Save to File.
Store in your repo under monitoring/dashboards/.

Option B — Fetch JSON from ConfigMaps (pre-provisioned dashboards)
kube-prometheus-stack stores dashboards as ConfigMaps. You can get them with kubectl.