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

