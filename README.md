Deployment of Kuard app in Kubernetes packed with HELM using ArgoCD as GitOps and with implemented logging functionality via Elasticsearch with Kubernetes Persistent Volumes
======
![image](https://i2.paste.pics/f9826a7a75cff7e9f771340faf63f57f.png)

This is an example project to showcase my skills in Kubernetes, HELM, ArgoCD. As an example  Kuard app has been chosen (Kuard - is a simple app usually used as a deploy example in k8s). Talking in particular I wanted to showcase my ability to deploy pods with k8s,  to set up the persistent volume, and to set up claims for volumes, also I want to showcase my skills in general to fully set up Kubernetes cluster as Highly-Available using Load-Balancer and my  skills to work with HELM Charts, and as a final note - I have used ArgoCD to implement GitOps technologies into my project. I decided also to additionally configure Horizontal Pod Autoscaler by using HPA included in Kuard with filebeat and metrics server, and on-top I also set up ES + Kibana for looking into logs from k8s
---
Main technological stack:

- Kubernetes
- HELM - > setting up everything, using public HELM charts and deploying them
- ArgoCD - > providing GitOps
- HAProxy - setting up Highly-Available Kubernetes

Additional technologies used(they were chosen as example for work, therefore I have only basic knowledge of them):

- Elasticsearch
- Kibana 

---
### Folder Structure of repository 
```
- infra --> Infrastructure folder where everything except Kuard and Horizontal Pod Autoscaler
- infra/pv --> Persistent volume folder which contains both configurations for PV and PVC 
- helm-charts/kuard --> Contains templates that were used by HELM for Kuard deployment
- helm-charts/values.yaml --> Value file for HELM Chart of Kuard
- helm-charts/templates/hpa.yaml --> Yaml file for setting up Horizontal Pod Autoscaler that were configured manually
- wa --> Folder that contains both HELM and usual YAML file for setting up Kuard app
- app-of-apps.yaml --> Configuration file using an app-of-apps method in ArgoCD to read repository
```

## Overview
To begin with, Kubernetes were set up in HA variant with 1 control plane node and 2 worker nodes, for setting up in HA variance - I have used HAProxy, which you could find in the main folder in this git (**haproxy.cfg**). After setting up everything and installing Calico into the cluster, for nodes to communicate I was ready to go. For VMs I have used Droplets in Digital Ocean with the following configuration (2 vCPUs 4GB / 80GB Disk on CentOS 8)
```
[root@ruslan-master-node ~]# kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
ruslan-master-node   Ready    control-plane   19d   v1.25.0
ruslan-worker-1      Ready    <none>          19d   v1.25.0
ruslan-worker-2      Ready    <none>          19d   v1.25.0
```
I went and found Kuard app - as an example of an App for deploying, for it is purposed I had to work also with values.yaml from Kuard Chart to set up everything correctly, and after that, I went to ArgoCD to deploy it and choose method using the app of apps to work with Argo, as I did not plan to have big amount of infrastructure to deploy, then I deployed Kuard using ArgoCD and synchronized with my main folder. 

![image](https://i2.paste.pics/850484b80ead2947de7cf150f7f82d5b.png)

After that I decided to add and configure HPA(Horizontal Pod Autoscaler). **HPA file located in deployment-of-app-using-kubernetes-helm-argocd-with-logging-functionality-and-hpa/helm-charts/kuard/templates/hpa.yaml**. But it would not work solely without Metrics Server - as to HPA we need to feed it with some logs, that he should use to work, Metrics server were deployed using HELM charts (here where Metrics Server is located https://github.com/kubernetes-sigs/metrics-server). **With that I configured HPA to scale up if Pod memory is used more than 80% and within 1-5 pods**.

**To sum up, until this point I have fully deployed and working Kuard App deployed using ArgoCD with the app of apps method, Horizontal Pod Autoscaler that were manually configured to scale if Pod memory used more than 80% and in 1-5 pods limits and Server Metrics deployed with open-source HELM chart.** It is fine but still I have room for more, as additional functionality and as usual main stack on K8s Cluster, I decided to implement logging functionality to gather it up and deploy into Kibana to view, I have used again HELM Charts, that were deployed by developers of Kibana and ES, and I deployed another app of apps in ArgoCD to look especially into infra folder in my git and to apply that HELM Charts.

![image](https://i2.paste.pics/0affe060bd2698eff3d68aae072f3926.png)

First I had to set up 1 Persistent Volume and Persistent Volume Claim in my K8S cluster, for Elasticsearch to work right ( it is one of the prerequisites ). **Yaml files, that were used to set PV and PVC are located in deployment-of-app-using-kubernetes-helm-argocd-with-logging-functionality-and-hpa/infra/pv/** which you can check and look at. Okay, than I deployed Elasticsearch into my git where ArgoCD pulled and deployed it into my cluster fully working.

![image](https://i2.paste.pics/2a9a5b5a8ddf2bced3215486d733a9a9.png)

This is good that we have Elasticsearch, but still, It can not get any logs, this is why I used filebeat (located in the infra folder in the repository). Filebeat gave me the opportunity to connect Elasticsearch and provide logging data, Filebeat was deployed manually using a pre-made public HELM chart.
![image](https://i2.paste.pics/2a9a5b5a8ddf2bced3215486d733a9a9.png)

And finally, I pushed HELM chart of Kibana also into my repository, where ArgoCD pulled it and deployed: 

![image](https://i2.paste.pics/6e20083ef8f6dd585da8e5c0869e10e7.png)

--- 

Here is how at the end ArgoCD looks with all components deployed and working and synchronized.
Please ignore external secrets deployed as I tried it to use just for testing purposes.
![image](https://i2.paste.pics/f9b4a2355250bcdd4cb2ffb2fb9f03bf.png)
