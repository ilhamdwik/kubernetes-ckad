# kubernetes-ckad

### Admission Controllers
```
k exec kube-apiserver-kubemaster1 -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```


### Validating Admission Controllers

### Mutating Admission Controllers


### API Version
```
k explain deployment
```

##### Kubectl Convert
```
kubectl convert -f <old-file> --output-version <new-api>
```