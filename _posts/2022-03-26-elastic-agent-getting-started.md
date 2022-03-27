https://www.elastic.co/guide/en/fleet/master/secure-connections.html

https://www.elastic.co/guide/en/elasticsearch/reference/8.1/configuring-stack-security.html#stack-security-certificates

https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html

https://www.elastic.co/guide/en/fleet/master/secure-connections.html#generate-fleet-server-certs

https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html

> Using Tarball Install instead of Debian package

1. Download and unzip the Elasticsearch tarball
2. Create a CA using elasticsearch-certutil
```

```

## Create Certificate Authority
This will be used to create certificates for Elasticsearch, Kibana, Fleet, and Agents



## Elasticsearch Install

## Kibana Install

## Fleet Server Install

Use this site: https://www.elastic.co/guide/en/fleet/master/secure-connections.html#generate-fleet-server-certs

Create a CA
Create fleet-server certs using the CA
Go to Kibana -> Fleet
Click create Fleet Server
- On Step 1. Be sure to click Create Policy
- Add the IP of the Fleet Server
  - In this case it will be https://192.168.235.130:8220
Install Elastic-Agent as Fleet Server
```
sudo ./elastic-agent install -f \
  --url=https://192.168.235.130:8220 \
  --fleet-server-es=https://192.168.235.130:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE2NDgzNDU0NjE0MDU6Q0J3YUJzck5UcGFKSlkwQ1oxcW9QQQ \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-es-ca-trusted-fingerprint=7a6333543099078bcdefac2656099d29ba004b8f84311cca3e2185a2f50e3a53 \
  --certificate-authorities=/home/elastic-user/elastic-agent-8.1.1-linux-x86_64/ca/ca.crt \
  --fleet-server-cert=/home/elastic-user/elastic-agent-8.1.1-linux-x86_64/fleet-server/fleet-server.crt \
  --fleet-server-cert-key=/home/elastic-user/elastic-agent-8.1.1-linux-x86_64/fleet-server/fleet-server.key
```
- Remeber to change the `certificate-authorities`, `fleet-server-cert`, and `fleet-server-cert-key`

If you installed the debian package use `enroll` instead of `install`, and you need to start the elastic-agent with `sudo service elastic-agent start





```
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.1.1-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.1.1-darwin-x86_64.tar.gz
```

Generate Fleet Server Certificates
```
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert -
-ca /etc/elasticsearch/certs/http_ca.crt --name fleet-server --ip 192.168.235.130 --pem
```

https://www.elastic.co/downloads/beats/packetbeat

```
[*] Login with:
    Username: elastic
    Password: K=35BI-HQ*TphKFy4o-K
```



```
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  2_wdD1=cP7qU27SnVHuL

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  7a6333543099078bcdefac2656099d29ba004b8f84311cca3e2185a2f50e3a53

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMSIsImFkciI6WyIxOTIuMTY4LjIzNS4xMzA6OTIwMCJdLCJmZ3IiOiI3YTYzMzM1NDMwOTkwNzhiY2RlZmFjMjY1NjA5OWQyOWJhMDA0YjhmODQzMTFjY2EzZTIxODVhMmY1MGUzYTUzIiwia2V5IjoiN0xYN3lIOEJ0S0RMX1VRMFgwRWs6VUZJUmpvV3VSZ3VFZ01OaDdSLU5wQSJ9

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.
```