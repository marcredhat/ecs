

1) Basic checks

```
openssl s_client -connect <controlplane>:443
```

2) Corrupt DNS cache

```
strings /var/db/nscd/hosts
```

Noticed that nscd had lots of incorrect entries so cleaned it:```
nscd --invalidate=hosts
```

3) Known rke2-ingress-nginx-controller issues and fixes
 
k exec -it  rke2-ingress-nginx-controller-... -n kube-system -- top

Check number of worker processes and CPU utilisation 

Known bug high-spec ECS machines with many CPU cores as 
nginx-ingress-controller miscalculates values s.a.  MaxWorkerOpenFiles.

The symptom of the above bug was abnormally high CPU on ECS machines with >= 32 vCPUs

Fix (adapt values to your env)

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: rke2-ingress-nginx-controller
  namespace: kube-system
data:
  max-worker-connections: "10000"
  max-worker-open-files: "10204"
  upstream-keepalive-connections: "100"
  keep-alive-requests: "100000000"
```


4) 
```
strace -e trace=network -r -f curl -k  -vvv --write-out '\nlookup:        %{time_namelookup}\nconnect:       %{time_connect}\nappconnect:    %{time_appconnect}\npretransfer:   %{time_pretransfer}\nredirect:      %{time_redirect}\nstarttransfer: %{time_starttransfer}\ntotal:         %{time_total}\n' https://vault.localhost.localdomain/v1/sys/seal-status
```


5) DNS resolution issues

```
wrk -t2 -c1000 -d180s -L -R30000 https://vault.localhost.localdomain/v1/sys/seal-status
=>  16151 timeouts; avg latency = 1.57m

Skipping name resolution by going directly to the vault-0 IP address:
wrk -t2 -c1000 -d180s -L -R30000 https://x.x.x.x:8200/v1/sys/seal-status
=> 0 timeout, avg latency = 4.22ms
```

https://www.nginx.com/blog/performance-testing-nginx-ingress-controllers-dynamic-kubernetes-cloud-environment/.


6) NSX-T creates distributed firewall rules based on the Network Policies created by the Private Cloud.

This include default deny rules which block the CDP Private Cloud installation.

Check NetworkPolicies
Check iptables on ECS server
k get NetworkPolicy -A 
