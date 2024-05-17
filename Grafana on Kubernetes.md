Grafana on KubernetesThis README provides a comprehensive guide on how to set up Grafana on a Kubernetes cluster.PrerequisitesA Kubernetes cluster (v1.16+)kubectl configured to interact with your clusterHelm (v3+)Steps1. Add Helm RepositoryFirst, add the Grafana Helm repository to your Helm configuration.shCopy codehelm repo add grafana https://grafana.github.io/helm-charts
helm repo update
2. Create a NamespaceCreate a namespace for Grafana to keep it organized within your cluster.shCopy codekubectl create namespace grafana
3. Install Grafana Using HelmInstall Grafana using the Helm chart. This will create all the necessary Kubernetes resources.shCopy codehelm install grafana grafana/grafana --namespace grafana
This command installs Grafana with default configurations. You can customize the installation by providing additional parameters.4. Access Grafana4.1. Retrieve the Admin PasswordRetrieve the auto-generated admin password for Grafana:shCopy codekubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
The default username is admin.4.2. Port Forward to Access Grafana LocallyYou can access Grafana through port forwarding:shCopy codekubectl port-forward --namespace grafana svc/grafana 3000:80
Now you can access Grafana at http://localhost:3000.5. Expose Grafana (Optional)To make Grafana accessible outside the cluster, you can set up an Ingress or change the Service type to LoadBalancer.5.1. Using LoadBalancer ServiceEdit the Grafana service to be of type LoadBalancer:shCopy codekubectl edit svc grafana -n grafana
Change the type from ClusterIP to LoadBalancer. Save and exit the editor. After a few moments, an external IP address will be assigned.5.2. Using IngressEnsure your cluster has an Ingress controller installed. Then, create an Ingress resource:yamlCopy codeapiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: grafana
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: &lt;your-domain&gt;
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 80
Apply the Ingress resource:shCopy codekubectl apply -f ingress.yaml
Replace &lt;your-domain&gt; with your actual domain name.6. Configure GrafanaAfter accessing Grafana, you can log in with the username admin and the password you retrieved earlier. You can then start configuring Grafana by adding data sources, setting up dashboards, and more.7. Persistent Storage (Optional)To ensure Grafana retains its state across restarts, configure persistent storage.7.1. Create a PersistentVolumeClaim (PVC)Create a PVC definition:yamlCopy codeapiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-storage
  namespace: grafana
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
Apply the PVC:shCopy codekubectl apply -f pvc.yaml
7.2. Update Helm Values to Use PVCUpdate the Helm values to use the PVC for Grafana's storage:shCopy codehelm upgrade grafana grafana/grafana --namespace grafana --set persistence.enabled=true --set persistence.existingClaim=grafana-storage
8. Uninstall GrafanaIf you need to uninstall Grafana, use the following command:shCopy codehelm uninstall grafana --namespace grafana
kubectl delete namespace grafana
This will remove all Grafana resources from your cluster.ConclusionYou have successfully set up Grafana on your Kubernetes cluster. You can now use Grafana to visualize and monitor your data. For more advanced configurations, refer to the Grafana documentation.
