/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew update && brew install kubectl && brew cask install docker minikube virtualbox

brew install minikube

brew cask install virtualbox

Sicherheit/Allgemein: Virtualbox erlauben

brew install docker

minikube start




// minikube docker-env
eval $(minikube docker-env)

Optional:
docker image pull richardchesterwood/k8s-fleetman-webapp-angular:release0-5

docker container run -p 8080:80 -d richardchesterwood/k8s-fleetman-webapp-angular:release0-5
Output: dd64b8473115b9dd95a6aa99355c844a20a9edd8fde00236b94ec59696130377

docker container ls
minikube ip
http://192.168.99.100:8080

docker container stop dd64
docker container rm dd64

___________________________________________________________________________

## Pods (short living, will often be restartet)
minikube status
kubectl get all
kubectl apply -f first-pod.yaml
kubectl describe pod webapp
kubectl exec webapp ls
kubectl -it exec webapp sh

___________________________________________________________________________

## Services
type: ClusterIP = private Service inside Kubernetis
      NotePort = exposed through the node > 30000

kubectl apply -f webapp-service.yaml

### Deploy with zero downtime
    add additional pod with new label (do it in same pod-file with --- as separator)
    select this new pod via label in the service

kubectl describe service fleetman-webapp
kubectl describe svc fleetman-webapp

kubectl get pods
kubectl get po

kubectl get po --show-labels

kubectl get po --show-labels -l release=0 // filter

kubectl apply -f .

___________________________________________________________________________

## ReplicaSets
kubectl delete po webapp-release-0-5
kubectl delete po --all  

kubectl describe replicaset webapp
kubectl describe rs webapp
___________________________________________________________________________

## Deployment
rolling updates with zero downtime and rollback, when problem

kubectl delete rs webapp

minReadySeconds

kubectl rollout status deployment webapp
kubectl rollout status deploy webapp
kubectl rollout history deploy webapp

Careful! Ony for emergencies. When using rollbacks.  
kubectl rollout undo deploy webapp
kubectl rollout undo deploy webapp --to-revision=4
___________________________________________________________________________

## Networking:

Service-kube-dns

## Namespaces: (partitioning resources pods/services) at example front-end, back-end

default-Namespace

kubectl get namespaces 
kubectl get ns 

kubectl get po -n kube-system
kubectl get all -n kube-system
kubectl describe svc kube-dns -n kube-system

### pod with mysql-database:
kubectl apply -f networking-tests.yaml

### shell into webapp-container:
kubectl exec -it webapp-858957ccc9-66t2h sh
nslookup database
apk update
apk add mysql-client
mysql -h database -uroot -ppassword fleetman
create table testtable (test varchar(255));
show tables;

## fully qualified domain names (FQDN)
Es wird automatisch im default namespace gesucht
__nslookup database__ sucht auch in __database.default.svc.cluster.local__
Wenn man in einem speziellen namespace suchen möchte, muss man den __FQDN__ angeben

## Increase VirtualBox memeory to 4 GB !!

## Microservice Architecture

### lösche gesamtes altes Deployment
kubectl delete -f .

## Logs for pod
kubectl logs <podname>
### follow logs
kubectl logs -f <podname>

## MongoDB
Per default, die Daten für MongoDB werden im Container gespeichert.
Wenn der Pod stirbt, sind die Daten verloren.

### Persitent Volume:
Für unser Beispiel speichern im lokalen Dateisystem (in der VirtualBox VM).
__volumeMounts__ im yaml

später -> __ebs-Volume__ = __awsElasticBlockStore__ auf AWS

Mit Doppelkick auf VirtualBox-Minicube kann man sich in die VM einloggen.
User: docker
Password: tcuser

### PersitentVolumeClaims
hilft dabei, nicht überall hostPath durch ebsPath ersetzen zu müssen, wenn man von einem Cloud-Provider zu einem anderen wechseln möchte.
At runtime the claim will be satified -> __binding__

# list of all persistent volumes:
kubectl get pv

# list of persistent volume claims:
kubectl get pvc

# AWS
Ein __Node__ ist ein physikalischer Server = __IC2-Instance__
Ein System = __cluster__ besteht aus mehreren __nodes__
Viele __kleine__ nodes

Es gibt einen __Master Node__ der für die Verwaltung der __nodes__ zuständig ist.

## kops
__Kubernetis Operations__
 
 
Instanz erstellen

add tag: (name bootstrap)
SSH My IP

 cp ../../Downloads/video-keypair.pem .
 chmod go-rwx video-keypair.pem        
 ssh -i video-keypair.pem ec2-user@52.59.238.184

 curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

neuer IAM User:
__Access key__ AKIAZCLXA45LVGAOY74B
__Secret_access key__ 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

 aws configure
 aws iam list-users

export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)

## Create Cluster State storage
aws s3api create-bucket \
    --bucket gaebler-goes-de-state-storage \
    --region eu-central-1 \
    --create-bucket-configuration LocationConstraint=eu-central-1

aws s3api put-bucket-versioning --bucket gaebler-goes-de-state-storage --versioning-configuration Status=Enabled

## Create the cluster
export NAME=fleetman.k8s.local
export KOPS_STATE_STORE=s3://gaebler-goes-de-state-storage

## Create Cluster Configuration
aws ec2 describe-availability-zones --region eu-central-1

kops create cluster \
    --zones eu-central-1a,eu-central-1b,eu-central-1c \
    ${NAME}

### wenn public key error:
ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa
no passphrase

kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub

### nur zum Nachschauen
export EDITOR=nano
kops edit cluster ${NAME}

## wichtig !
kops edit ig nodes --name ${NAME}
__ig__ = Instance Group

machineType: t2.micro: reicht nicht für ELK/Grafana nicht verändern
maxSize: 5
minSize: 3

kops get ig nodes --name ${NAME}

### Edit Master
machineType: t2.micro : reicht nicht für ELK! nicht verändern

## Run Cluster
kops update cluster ${NAME} --yes
kops validate cluster

## Provision SSD drives

### Wo läuft welcher Pod?
get pods -o wide

## Delete the cluster
kops delete cluster --name ${NAME} --yes

## Restart the cluster
start bootstrap instance
__Description:__ Security Group launch-wizard.. __Inbound__ tab edit instance: set to MyIP again
ssh into bootstrap
history | grep export
redo the __export fleetman.k8s.local__ and __KOPS_STATE_STORE...__ commands via !<linenumber>

__ctrl r__ to search for previous commands
kops c 

kubectl apply -f .


### Pods should be __stateless__
The queue-pod has __data__, you can´t __replicate__ it, because then the data will be split and chaos will occur.
A solution would be to use the AWS-Apache-Queue and let the provider make sure, the queue is fault-tolerant.
__question:__ what do I do with pods, that store data? 
__answer:__ Replicate the mongodb
__Youtube:__ dickchesterwood 2018

## Logging a Cluster - ELK = ElasticStack
### ElasticSearch
distributed search and analytics service (search engine based on __apache lucine__)
### Logstash or Fluentd
__Fluentd__ pulls logs from the Docker-___Containers___ = pods (can pull from many sources) and sends them to __Elasticsearch__
### Kibana
Allows you to visualizes __ElasticSearch__ data

## Installing ELK
You can use the Stack via services from the cloud-provider
We will install them as docker-containers = pods for learning

https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch

__DaemonSet__ kind of replicaSet, that __needs to run on every node__
__StatefulSet__ kind of replicaSet, where the pods get a specific name defined in the yaml for clustering
__volumeclaimTemplate__ points to ebs volume for ElasticSearch-Data-Storage

ELK-Stack is deployed to __kube-system__ namespace
kubectl get po -n kube-system

kubectl get svc -n kube-system
kibana-logging          LoadBalancer   100.69.194.17   ac6090d9af8694d0aa3f9b3e56f8a9e4-606622198.eu-central-1.elb.amazonaws.com   5601:31220/TCP   4m58s

kubectl describe svc kibana-logging -n kube-system
kibana-logging          LoadBalancer   100.69.194.17   ac6090d9af8694d0aa3f9b3e56f8a9e4-606622198.eu-central-1.elb.amazonaws.com   5601:31220/TCP   4m58s

http://ac6090d9af8694d0aa3f9b3e56f8a9e4-606622198.eu-central-1.elb.amazonaws.com/5601

## zeigt die Kibana-Website

index pattern: logstash
time field: @timestamp

Discover 
Set Filters
Save filter
Visualize
Save Visualition
Dashboard add visualation

## Monitor a Cluster

### Helm Package Manager
https://github.com/helm/helm
https://helm.sh

copy link to binary
on the bootstrap-instance:
wget https://get.helm.sh/helm-v3.1.1-linux-386.tar.gz
tar zxvf helm-v3.1.1-linux-386.tar.gz 
sudo mv linux-386/helm /usr/local/bin/
rm helm-v3.1.1-linux-386.tar.gz 
rm -rf ./linux-386/

helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm search repo stable

helm install my-special-installation stable/mysql --set mysqlPassword=password
helm ls
helm delete my-special-installation

## Modify Service/Pod on the fly
kubectl edit <name>

### Prometheus
https://prometheus.io
used for monitoring
we are using __prometheus-operator__ which is fully configured

kubectl create namespace monitoring
helm install monitoring stable/prometheus-operator --namespace monitoring

### Grafana
https://grafana.com
used as User-Interface

kubectl get all -n monitoring
(find the grafana-service)
kubectl edit -n monitoring service/monitoring-grafana
(set ClusterIP to LoadBalancer)
Open LoadBalancer-Url in Browser

username: admin
password: prom-operator

## The Alert Manager

kubectl get svc -n monitoring

### Visit any service in a cluster
The __api-gateway__ is accessable via a public DNS name on https://

Allow access without certificate.

### get username and password:
kubectl config view --minify

https://api-fleetman-k8s-local-tkmafs-1362626428.eu-central-1.elb.amazonaws.com/api/v1/namespaces

https://api-fleetman-k8s-local-tkmafs-1362626428.eu-central-1.elb.amazonaws.com/api/v1/namespaces/monitoring/services/alertmanager-operated:9093/proxy

__works with simple services!__

### Setting up a Slack Channel

create channel #alert
create app

https://hooks.slack.com/services/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

### Configuring AlertManager
alertmanager.yaml

__group-interval__ all alerts with the same name in this interval are grouped into a single notification
__group-wait__ wait this interval, if this alert is resolved (don´t show glitches)
__repeat_interval__ any alarm, that is not resolved will be repeated this interval

### Applying Config

kubectl get secret

kubectl get secret -n monitoring alertmanager-monitoring-prometheus-oper-alertmanager -o json

echo Z2xvYmFsOgogIHJlc29sdmVfdGltZW91dDogNW0KcmVjZWl2ZXJzOgotIG5hbWU6ICJudWxsIgpyb3V0ZToKICBncm91cF9ieToKICAtIGpvYgogIGdyb3VwX2ludGVydmFsOiA1bQogIGdyb3VwX3dhaXQ6IDMwcwogIHJlY2VpdmVyOiAibnVsbCIKICByZXBlYXRfaW50ZXJ2YWw6IDEyaAogIHJvdXRlczoKICAtIG1hdGNoOgogICAgICBhbGVydG5hbWU6IFdhdGNoZG9nCiAgICByZWNlaXZlcjogIm51bGwi | base64 -d 

kubectl delete secret alertmanager-monitoring-prometheus-oper-alertmanager -n monitoring
kubectl create secret -n monitoring generic alertmanager-monitoring-prometheus-oper-alertmanager --from-file=alertmanager.yaml

kubectl logs -n monitoring alertmanager-monitoring-prometheus-oper-alertmanager-0 -c alertmanager

__master-node__ sends commands to the nodes (at example: rescheduling a dead pod). So the __tempory loss of a master__ is no problem

you can create a __highly available cluster__

## Instance Models
__t2__ cheap, baseline level low with ability to burst above it. __credit balance__

Course has been updated. I try to get the update

## Requests

Limits for the Scheduler (essential)
Scheduler will not create pod, when the requests can´t be satisfied

__memory request__
__cpu request__

yaml
containers:
 resources:
    requests:
        memory:

kubectl get all --all-namespaces
kubectl describe node minikube

## Limits

Runtime Limits (safety net, optional)
If the __memory limits__ are exceeded, the container will be killed.
The pod will remain. The container will be restartet 

If the __cpu limits__ are exceeded, the cpu will be "clamped".
The container will continue to run

yaml
containers:
 resources:
    limits:
        memory: 500Mi
        cpu: 200m

__OOMKilled__ = OutOfMemoryKilled

## addons

minikube addons list
minikube addons enable  metrics-server

## Metrics (metric server)

### needs at leat a minute to gather data

kubectl top pod
kubectl top node

kubectl get all -n kube-system

## Horizontal Pod Autoscaling

### Improving the capacity of a service by creating more instances

## Vertical Scaling

### Improving the capacity of a service by making instance more powerfull

__HPA__ = horizontal autoscaler

### scale api-gateway when cpu usage is 4 * resources:requests:cpu = 200m
### min: 1 replica-set max: 4 replica-sets (beware of DOS-Attack)
kubectl autoscale deployment api-gateway --cpu-percent 400 --min 1 --max 4

kubectl get hpa
kubectl describe hpa

kubectl get hpa api-gateway -o yaml

