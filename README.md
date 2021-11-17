# intro-to-k8s

Assets used for the production of the Introduction to Kubernetes course on Cloud Academy
<a target="_blank" href="https://cloudacademy.com/course/introduction-to-kubernetes/">https://cloudacademy.com/course/introduction-to-kubernetes</a>

## Commands
The following commands are used in the Introduction to Kubernetes course, organized by lesson.

### Pods
```bash
kubectl get pods
cd src
kubectl create -f 1.1-basic_pod.yaml
kubectl get pods
kubectl describe pod mypod | more
kubectl delete pod mypod
kubectl create -f 1.2-port_pod.yaml
kubectl describe pod mypod | more
curl 192.168.###.###:80

# Replace ###.### with the IP address octets from the describe output)
# This command will time out (see the next lesson to understand why)

kubectl describe pod mypod | more
kubectl delete pod mypod
kubectl create -f 1.4-resources_pod.yaml
kubectl describe pod mypod | more

# Note: kubectl will accept the singular or plural form of resource kinds. For example kubectl get pods and kubectl get pod are equivalent.
```

### Services
```bash
kubectl create -f 2.1-web_service.yaml
kubectl get services
kubectl describe service webserver
kubectl describe nodes | grep -i addresses -A 1
curl 10.0.0.100:3####

# replace #### with the actual port digits
```

### Multi-Container Pods
```
kubectl create -f 3.1-namespace.yaml
kubectl create -f 3.2-multi_container.yaml -n microservice
kubectl get -n microservice pod app
kubectl describe -n microservice pod app
kubectl logs -n microservice app counter --tail 10
kubectl logs -n microservice app poller -f
```

### Service Discovery
```bash
kubectl create -f 4.1-namespace.yaml
kubectl create -f 4.2-data_tier.yaml -n service-discovery
kubectl get pod -n service-discovery
kubectl describe service -n service-discovery data-tier
kubectl create -f 4.3-app_tier.yaml -n service-discovery
kubectl create -f 4.4-support_tier.yaml -n service-discovery
kubectl get pods -n service-discovery
kubectl logs -n service-discovery support-tier poller -f
```

### Deployments
```bash
kubectl create -f 5.1-namespace.yaml
kubectl create -n deployments -f 5.2-data_tier.yaml -f 5.3-app_tier.yaml -f 5.4-support_tier.yaml
kubectl get -n deployments deployments
kubectl -n deployments get pods
kubectl scale -n deployments deployment support-tier --replicas=5
kubectl -n deployments get pods
kubectl delete -n deployments pods support-tier-... support-tier-... --wait=false

# Use tab completion to display the possible values to replace ...

watch -n 1 kubectl -n deployments get pods
kubectl scale -n deployments deployment app-tier --replicas=5
kubectl -n deployments get pods
kubectl describe -n deployments service app-tier
```

### Autoscaling
```bash
# metrics-server is pre-installed
# kubectl create -f metrics-server

watch kubectl top pods -n deployments
kubectl create -f 6.1-app_tier_cpu_request.yaml -n deployments
kubectl apply -f 6.1-app_tier_cpu_request.yaml -n deployments
kubectl get -n deployments deployments app-tier
kubectl create -f 6.2-autoscale.yaml -n deployments
watch -n 1 kubectl get -n deployments deployments app-tier

# This can take up to 13 minutes

kubectl api-resources
kubectl describe -n deployments hpa
kubectl get -n deployments hpa
kubectl edit -n deployments hpa
watch -n 1 kubectl get -n deployments deployments app-tier
```

### Rolling Updates and Rollbacks
```bash
kubectl delete -n deployments hpa app-tier
kubectl edit -n deployments deployment app-tier
watch -n 1 kubectl get -n deployments deployments app-tier
kubectl edit -n deployments deployment app-tier
kubectl rollout -n deployments status deployment app-tier
tmux
kubectl edit -n deployments deployments app-tier (left terminal)
kubectl rollout -n deployments status deployment app-tier (right terminal)
kubectl rollout -n deployments pause deployment app-tier (left terminal)
kubectl get deployments -n deployments app-tier (left terminal)
kubectl rollout -n deployments resume deployment app-tier (left terminal)
kubectl rollout -n deployments undo deployment app-tier
kubectl scale -n deployments deployment app-tier --replicas=1
```

### Probes
```bash
kubectl create -f 7.1-namespace.yaml
kubectl create -f 7.2-data_tier.yaml -n probes
kubectl get deployments -n probes -w
kubectl create -f 7.3-app_tier.yaml -n probes
kubectl get -n probes deployments app-tier -w
kubectl get -n probes pods
kubectl logs -n probes app-tier-... | cut -d' ' -f5,8-11

# Use tab completion to display the possible values to replace ...
```

### Init Containers
```bash
kubectl apply -f 8.1-app_tier.yaml -n probes
kubectl describe pod -n probes app-tier...

# Use tab completion to display the possible values to replace ...

kubectl logs -n probes app-tier-... await-redis

# Use tab completion to display the possible values to replace ...
```

### Volumes
```bash
kubectl -n deployments logs support-tier-... poller --tail 1

# Use tab completion to display the possible values to replace ...

kubectl exec -n deployments data-tier-... -it -- /bin/bash

# Use tab completion to display the possible values to replace ...

kill 1 # to halt process

kubectl -n deployments get pods
kubectl -n deployments logs support-tier-... poller --tail 1

# Use tab completion to display the possible values to replace ...

#Note: It takes around a couple of minutes for the effects of the restart to settle. The poller will stop updating and report the last value before restarting until it can reach the new data tier value. Try again after a minute if you don't see a relatively small value

kubectl create -f 9.1-namespace.yaml
aws ec2 describe-volumes --region=us-west-2 --filters="Name=tag:Type,Values=PV" --query="Volumes[0].VolumeId" --output=text
vol_id=$(aws ec2 describe-volumes --region=us-west-2 --filters="Name=tag:Type,Values=PV" --query="Volumes[0].VolumeId" --output=text)
sed -i "s/INSERT_VOLUME_ID/$vol_id/" 9.2-pv_data_tier.yaml
kubectl create -n volumes -f 9.2-pv_data_tier.yaml -f 9.3-app_tier.yaml -f 9.4-support_tier.yaml
kubectl describe -n volumes pvc
kubectl describe -n volumes pod data-tier-...

# Use tab completion to display the possible values to replace ...

kubectl logs -n volumes support-tier-... poller --tail 1

# Use tab completion to display the possible values to replace ...

# Note: It takes a few minutes for all of the readiness checks to pass and for the counter to start incrementing. If you don't see a counter value output then try again after a minute or two.

kubectl delete -n volumes deployments data-tier
kubectl get -n volumes pods
kubectl create -n volumes -f 9.2-pv_data_tier.yaml
kubectl logs -n volumes support-tier-... poller --tail 1

# Use tab completion to display the possible values to replace ...
```

### ConfigMaps and Secrets
```bash
kubectl create -f 10.1-namespace.yaml
kubectl create -n config -f 10.2-data_tier_config.yaml -f 10.3-data_tier.yaml
kubectl exec -n config data-tier-... -it -- /bin/bash

# Use tab completion to display the possible values to replace ...

cat /etc/redis/redis.conf
redis-cli CONFIG GET tcp-keepalive
exit
kubectl edit -n config configmaps redis-config
kubectl exec -n config data-tier-... -- redis-cli CONFIG GET tcp-keepalive

# Use tab completion to display the possible values to replace ...

kubectl rollout -n config restart deployment data-tier
kubectl exec -n config data-tier-... -- redis-cli CONFIG GET tcp-keepalive

# Use tab completion to display the possible values to replace ...

kubectl create -f 10.4-app_tier_secret.yaml -n config
kubectl describe -n config secret app-tier-secret
kubectl edit -n config secrets app-tier-secret
kubectl create -f 10.5-app_tier.yaml -n config
kubectl exec -n config app-tier-... -- env

# Use tab completion to display the possible values to replace ...
```
