# Notes
Just some personal random notes

```
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

# If need time to exec -it in a container to debug
```
    spec:
      containers:
      - name: my-container
        image: my-image
        command: ["/bin/sleep", "infinity"]  # Sleep indefinitely, change the command as needed
```

# Recursive sed
```
find /some/file/path/ -type f -exec sed -i '/clusterIP/,+2d' {} \;
```

# Get manifests
```
kubectl get cm -A| grep -v NAMESPACE | while read line; do
  pod_name=$(echo $line | awk '{print $2}' ) \
  name_space=$(echo $line | awk '{print $1}' ); \
  kubectl get cm $pod_name -n $name_space -o yaml > $pod_name-cm.$(date +'%Y%m%d-%H%M').yaml
done

kubectl get sts -A| grep -v NAMESPACE | while read line; do
  pod_name=$(echo $line | awk '{print $2}' ) \
  name_space=$(echo $line | awk '{print $1}' ); \
  kubectl get sts $pod_name -n $name_space -o yaml > $pod_name-sts.$(date +'%Y%m%d-%H%M').yaml
done

kubectl get svc -A| grep -v NAMESPACE | while read line; do
  pod_name=$(echo $line | awk '{print $2}' ) \
  name_space=$(echo $line | awk '{print $1}' ); \
  kubectl get svc $pod_name -n $name_space -o yaml > $pod_name-svc.$(date +'%Y%m%d-%H%M').yaml
done

```

# Delete non running pods
```
kubectl get pods -A -o wide | grep RUNNING -v| while read line; do
  pod_name=$(echo $line | awk '{print $2}' ) \
  name_space=$(echo $line | awk '{print $1}' ); \
  kubectl delete pods $pod_name -n $name_space --grace-period=0 --force
done

```

# Check cluster
```
echo;echo |awk '{print "|---PodNum---|------IP------|--------CPU------|-----Memory----|-----storage----|"}';for i in `kubectl get no --no-headers -o name`;do kubectl describe $i > /tmp/tmp;printf "%-13s" "$(cat /tmp/tmp|sed -n '/^Non-terminated Pods/s/^.*(\(.*\))$/\1/p')";printf " %-14s" $(echo $i|awk -F'/' '{print $NF}');cat /tmp/tmp | awk '/^Allocated resources/,/^Event/{if($1=="cpu"||$1=="ephemeral-storage"||$1=="memory"){printf "%-1s %-7s %-7s"," ",$2,$3}}';echo;rm -f /tmp/tmp;done
```

# Change pod limit
```
echo;echo -n 'Current';sed -n '/max-pod/p' /etc/systemd/system/kubelet.service;read -p "Input new max-pods=" a ;echo $a|[ -z "sed -n '/^[0-9][0-9]*$/p'" ] && echo 'Invalid input, Ctrl+C quit' && sleep 9999 || echo 'running...' ; sed -i '/max-pods/s/=[0-9]*/='$a'/' /etc/systemd/system/kubelet.service;sleep 1;systemctl daemon-reload && sleep 1;systemctl restart kubelet && echo 'Adjusted successfully' && cat /etc/systemd/system/kubelet.service | grep max-pod
```

```
echo;echo -n 'Current ';sed -n '/maxPods/p' /var/lib/kubelet/config.yaml;read -p "Input new maxPods:" a ;echo $a|[ -z "sed -n '/^[0-9][0-9]*$/p'" ] && echo 'Invalid input, Ctrl+C quit' && sleep 9999 || echo 'running...' ; sed -i '/maxPods/s/: [0-9]*/: '$a'/' /var/lib/kubelet/config.yaml;sleep 1;systemctl daemon-reload && sleep 1;systemctl restart kubelet && echo 'Adjusted successfully' && cat /var/lib/kubelet/config.yaml | grep maxPods
```

# check recently restarted pods
```
kubectl get po -A | egrep -v '[0-9]d|[0-9]h|[2-9][0-90]m|default'
```

