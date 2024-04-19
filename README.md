# gitops-tkgs

This repo can be used to test a GitOps approach with ArgoCD and a vSphere with Tanzu Supervisor cluster. ArgoCD will be installed in a TKG workload cluster as described below. The Supervisor cluster and vSphere Namespace will be connected as a target. 

## Install ArgoCD on a TKG cluster

1. Install ArgoCD on TKG cluster    
kubectl create ns argocd  
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Note: be aware of docker rate limits, you might want to use an imagepullsecret for the redis pod (create secret first)  
kubectl -n argocd patch serviceaccount argocd-redis -p '{"imagePullSecrets": [{"name": "regcred"}]}’

1. Change to service type LoadBalancer  
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

1. Install argocd cli on your client  
brew install argocd

1. Adjust argocd configmap with resource exclusions and inclusions for supervisor usage, see example [here](argocd-config/argocd-cm.yaml)  
Kubectl -n argocd edit cm argocd-cm

1. Change argo default pw  
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d  
argocd login x.x.x.x  
argocd account update-password

## Add the Supervisor cluster to ArgoCD

1. Logon to your Supervisor Cluster as administrator and create a service account in a vSphere Namespace (argocd system-namespace)  
kubectl create serviceaccount argocd-sa -n ese

2. Create a rolebinding within the vSphere Namespace that you want to use as a target  
Kubectl create rolebinding argo-edit-binding --clusterrole=edit --serviceaccount=ese:argocd-sa -n usercon

Note: To onboard additional vSphere Namespaces, create the same rolebinding in the new namespace and add the namespace to the managed namespaces under the ArgoCD cluster configuration

3. Add the Supervisor Cluster to ArgoCD via argocd CLI  
argocd cluster add x.x.x.x --service-account argocd-sa --system-namespace ese --namespace usercon

After adding the Supervisor cluster to ArgoCD you can decide if you want to create the ArgoCD applicaitons via the UI or to use the manifests [here](argocd-config)

Argo CD repository, configmap and application manifests in argocd-config/  
TKC manifest in tkc-config/  
VM Services manifest in vmservice/  

Rersources mentioned in this blog post https://beyondelastic.com/2021/07/29/gitops-with-argo-cd-on-vsphere-with-tanzu/ can be found in old/. However, try to follow the new approach mentioned above. 
