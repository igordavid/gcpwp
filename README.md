# gcpwp
Wordpress on GCP

                         Users           Users           Users           Users
                         
                           |                 |              |                |
                           
                           
                                                 GCP CDN
                                           
                                                |       |
                                           
                                           
                                            GCP LOAD BALANCER
                                            
                                            
                                                |       |
                                                
                                                
                                     GCP KUBERNETES CLUSTER (Autoscaled)
                                             
                                             
                                                |       |
                                                
                                                
                                         MARIADB ON PERSISTENT DISK
                                           
                                           
                                           
                                           

Instructions:


$ CLUSTER_VERSION=$(gcloud beta container get-server-config --region us-west2 --format='value(validMasterVersions[0])')
$ export CLOUDSDK_CONTAINER_USE_V1_API_CLIENT=fa
$ gcloud beta container clusters create repd \
  --cluster-version=${CLUSTER_VERSION} \
  --machine-type=n1-standard-1 \
  --region=us-west2 \
  --num-nodes=1 \
  --node-locations=us-west2-b,us-west2-c

$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
$ kubectl create serviceaccount tiller --namespace kube-system
$ kubectl create clusterrolebinding tiller-cluster-rule 

$ helm init --service-account=tiller
$ helm init --service-account=tiller


$ kubectl apply -f - <<EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: repd-west2-b-c
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: regional-pd
  zones: us-west2-b, us-west2-c
EOF

storageclass.storage.k8s.io "repd-west2-b-c" created



helm install --name wp-repd \
  --set persistence.storageClass=repd-west2-b-c \
  stable/wordpress \
  --set mariadb.persistence.storageClass=repd-west2-b-c


https://cloud.google.com/monitoring/kubernetes-engine/prometheus:


$ curl -sSO "https://storage.googleapis.com/stackdriver-prometheus-documentation/rbac-setup.yml 
$ get_helm.sh rbac-setup.yml
$ curl -sSO "https://storage.googleapis.com/stackdriver-prometheus-documentation/prometheus-service.yml"
$ kubectl apply -f prometheus-service.yml
$ kubectl get deployment,service -n stackdriver


NEXT STEPs:

Add Load balancing  (DONE)
Add autoscaling (DONE)
Add CDN (DONE with ERRORS)
Add terraform
