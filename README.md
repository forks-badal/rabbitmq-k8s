# rabbitmq-k8s
RabbitMQ >=3.7 cluster using Kubernetes >=1.5 on [GKE](https://cloud.google.com/kubernetes-engine/)

* Replace `staging-project-id` and `production-project-id` with your projects in the next steps.
* Clone the repo and configure your container images:
```
$ git clone https://github.com/goll/rabbitmq-k8s.git && cd rabbitmq-k8s/
$ sed -i -e 's/{{PROJECT-ID}}/staging-project-id/' -e "s/{{COMMIT}}/$(git log -1 --pretty=%h)/" kube/stateful.set.staging.yaml
$ sed -i -e 's/{{PROJECT-ID}}/production-project-id/' -e "s/{{COMMIT}}/$(git log -1 --pretty=%h)/" kube/stateful.set.yaml
```

* Define the project for gcloud commands:
```
$ export PROJECT_ID="staging-project-id"
or
$ export PROJECT_ID="production-project-id"
```

* Build and upload a new image:
```
$ docker build -t eu.gcr.io/${PROJECT_ID}/rabbitmq:$(git log -1 --pretty=%h) docker/
$ gcloud docker --project=${PROJECT_ID} -- push eu.gcr.io/${PROJECT_ID}/rabbitmq:$(git log -1 --pretty=%h)
```

* Deploy staging:
```
# Check if the cluster exists:
$ gcloud container clusters list --project=${PROJECT_ID}

# Create cluster if it doesn't exist:
$ gcloud container clusters create rabbitmq-staging-cluster --num-nodes=1 --machine-type=n1-standard-1 --zone=europe-west1-b --additional-zones=europe-west1-c,europe-west1-d --project=${PROJECT_ID}

# Get cluster credentials
$ gcloud container clusters get-credentials rabbitmq-staging-cluster --zone=europe-west1-b --project=${PROJECT_ID}
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)

$ ./setup-staging.sh
```

* Deploy production:
```
# Check if the cluster exists:
$ gcloud container clusters list --project=${PROJECT_ID}

# Create cluster if it doesn't exist:
$ gcloud container clusters create rabbitmq-production-cluster --num-nodes=1 --machine-type=n1-standard-1 --zone=europe-west1-b --additional-zones=europe-west1-c,europe-west1-d --project=${PROJECT_ID}

# Get cluster credentials
$ gcloud container clusters get-credentials rabbitmq-production-cluster --zone=europe-west1-b --project=${PROJECT_ID}
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)

$ ./setup-production.sh
```

* Delete staging:
```
$ kubectl get ns rmq-staging && kubectl delete ns rmq-staging
$ gcloud container clusters delete rabbitmq-staging-cluster --zone=europe-west1-b --project=${PROJECT_ID}
```

* Delete production:
```
$ kubectl get ns rmq && kubectl delete ns rmq
$ gcloud container clusters delete rabbitmq-production-cluster --zone=europe-west1-b --project=${PROJECT_ID}
```

* Delete disks:
```
$ gcloud compute disks list --project=${PROJECT_ID}
$ gcloud compute disks delete DISK_NAME --zone=ZONE --project=${PROJECT_ID}
```
