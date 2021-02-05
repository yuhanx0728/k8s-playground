# k8s-playground

## Working with pods
```
kubectl apply -f pod.yml
kubectl get pods --watch
kubectl get pods hello-pod -o yaml
kubectl describe pods hello-pod
kubectl exec hello-pod -- ps aux // execute commands in hello-pod
kubectl exec -it hello-pod -- sh // log into hello-pod
apk add curl // inside hello-pod
curl localhost:8080 // inside hello-pod
exit // inside hello-pod
kubectl logs hello-pod
kubectl delete -f pod.yml
```
