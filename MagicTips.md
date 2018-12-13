

## My favourite Aliases
```sh
alias k=kubectl
alias k=kubectl
alias ks="kubectl -n kube-system"
kn(){ kubectl -n $ns "$@";}
ka(){ kubectl "$@" --all-namespaces;}
export KUBECONFIG=~/kube-config/devcluster.conf 
kn get secret alertmanager-main -o "jsonpath={.data['alertmanager\.yaml']}" | base64 -D
```
## Alertmanager updates on prometheus operator; I always forget this one,
```sh
kn create secret generic alertmanager-main --from-literal=alertmanager.yaml="$(< alertmanager.yaml)" --dry-run -oyaml | 
kn replace secret --filename=-
```

## Virtually hog memory to test memory alerts
```sh
dd if=/dev/zero of=/tmp/hd-fillup.zeros bs=1G count=50
fallocate -l 50G file 
fallocate -l 50G big_file
truncate -s 50G big_file
dd of=bigfile bs=1 seek=50G count=0
df -h . ; dd of=biggerfile bs=1 seek=5000G count=0 ; ls -log biggerfile ; df -h .
memhog
```

## Quickly set proxy on VM with no Vim
```sh
cat > /etc/apt/apt.conf << EOF
Acquire::http::proxy "http://www-proxy.us.oracle.com:80/";
Acquire::https::proxy "https://www-proxy.us.oracle.com:80/";
Acquire::ftp::proxy "ftp://www-proxy.us.oracle.com:80/";
EOF
```

## init container for Checking Whether grafana is up or not

```sh
echo \"waiting for endpoints...\"; 
while true; 
	set endpoints \
		(curl -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
		--header \"Authorization: Bearer \"(cat /var/run/secrets/kubernetes.io/serviceaccount/token) \
		https://kubernetes.default.svc/api/v1/namespaces/monitoring/endpoints/grafana); 

	echo $endpoints | jq \".\"; 

	if test \
		(echo $endpoints | jq -r \".subsets[]?.addresses // [] | length\") -gt 0; 
		exit 0; 
	end; 

	echo \"waiting...\";
	sleep 1; 
end"
```

## Job which seeds Grafana Dashboard

```sh
for file in *-datasource.json ; do
  if [ -e "$file" ] ; then
    echo "importing $file" &&
    curl --silent --fail --show-error \
      --request POST http://admin:admin@grafana:3000/api/datasources \
      --header "Content-Type: application/json" \
      --data-binary "@$file" ;
    echo "" ;
  fi
done ;

for file in *-dashboard.json ; do
  if [ -e "$file" ] ; then
    echo "importing $file" &&
    ( echo '{"dashboard":'; \
      cat "$file"; \
      echo ',"overwrite":true,"inputs":[{"name":"DS_PROMETHEUS","type":"datasource","pluginId":"prometheus","value":"prometheus"}]}' ) \
    | jq -c '.' \
    | curl --silent --fail --show-error \
      --request POST http://admin:admin@grafana:3000/api/dashboards/import \
      --header "Content-Type: application/json" \
      --data-binary "@-" ;
    echo "" ;
  fi
done
```

## Create metric using file based exporter
```sh
touch /tmp/metrics-temp
while true
  for directory in (du --bytes --separate-dirs --threshold=100M /mnt)
    echo $directory | read size path
    echo "node_directory_size_bytes{path=\"$path\"} $size" \
      >> /tmp/metrics-temp
  end
  mv /tmp/metrics-temp /tmp/metrics
  sleep 300
end
```

## Searching a User in LDAP
```sh
ldapsearch -o ldif-wrap=no -h ldap.myorg.com -z 0 -S cn -LLL -x -b "dc=myorg,dc=com" "phonenumber=1206xxxzzzz"
```