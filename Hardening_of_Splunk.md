
# Hardening of Splunk

### Universal Forwarders
- [ ] Install UF as non-root user and keep it up-to-date 
- [ ] Disable listening Splunkd REST port (TCP/8089) in `server.conf`
	- [ ] Tip: Deploy this as an app from Deployment Server 
```
[httpServer] 
disableDefaultPort = true
```

- [ ] Use Deployment Server to configure inputs/outputs • Also, forward _internal and _audit to your indexers via `outputs.conf`
- https://docs.splunk.com/Documentation/Splunk/latest/DistSearch/Forwardsearchheaddata


Deploy base app from the OS to get vis from all the SPlunk Copm



## Heavy Forwarders

Follow UF Guidelines PLUS… 
- [ ] Disable unnecessary services (web, kvstore, rest) 
- [ ] Deploy passwords securely (e.g. via `ansible-vault`) 
- [ ] Do not put plaintext credentials in git repos Manage credentials using a secret manager 
	- [ ] `TA-VaultSync` will pull creds for any `passwords.conf`-based TA from Hashicorp Vault 
	- [ ] Checkout TRU1240C Automated Credential Synchronization with Hashicorp Vault 
- [ ] Consider using Docker Swarm to run HFs for high-availability and built-in security 
	- https://github.com/splunk/docker-swarm-splunk-hf

## Deployment Servers
- [ ] Use pass4symmkey to authenticate clients
- https://www.duanewaddle.com/splunk-pass4symmkey-for-deployment-client-deployment-server/ 
- [ ] Configure a non-standard port for splunkd connections in `web.conf`
```
[settings] 
mgmtHostPort = 0.0.0.0:38089
```
- [ ] Use a Web Application Firewall (WAF) in front of the Deployment Server
```
UserAgent ^Splunk 
HTTP Method = POST
```
- [ ] Consider separate DS for internal vs external clients

- [ ] Limit Splunk web access to internal networks via `firewall|iptables|security` group 
	- [ ] And if possible, splunkd access 
- [ ] Consider client certificates Consider MFA for SSH and web access 
- [ ] Change default splunk certificates ( Splunk Ships with the same Certs so change them)
-  https://wiki.splunk.com/images/f/fb/SplunkTrustApril-SSLipperySlopeRevisited.pdf 
- [ ] Install Splunk as non-root user and keep it up-to-date

## Indexers
- [ ] Configure pass4symmkey on indexer clusters 
- [ ] Enable SSL for HTTP Event Collector (HEC) and Splunk2Splunk (TCP/9997)
- [ ] Disable splunk web 
- [ ] Use a dedicated volume for $SPLUNK_DB limited to the splunk user 
- [ ] Enable data integrity control in default stanza of `indexes.conf` 
- https://docs.splunk.com/Documentation/Splunk/latest/Security/Dataintegritycontrol#Configure_data_integrity_control
- [ ] Limit Splunk2Splunk (TCP/9997) network connectivity to expected networks 
	- acceptFrom = in inputs.conf 
- [ ] SSH access 
	- [ ] Limit access, consider MFA or ZeroTrust solutions (e.g. ScaleFT) 
	- [ ] Use LDAP for authentication or if using SSH keys, rotate periodically 
- [ ] Enable indexer acknowledgment 
	-  https://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/Protectagainstthelossofin-flightdata 
	-  https://docs.splunk.com/Documentation/Splunk/latest/Data/AboutHECIDXAck 
- [ ] Change default splunk certificates 
	- [ ] https://wiki.splunk.com/images/f/fb/SplunkTrustApril-SSLipperySlopeRevisited.pdf

## Search Heads

- [ ] Limit search bundle max lookup size in `distsearch.conf`
```
[replicationSettings] 
excludeReplicatedLookupSize = 10
```
- [ ]  Limit max memory searches can consume in `limits.conf` 
```
[search] enable_memory_tracker = true search_process_memory_usage_percentage_threshold = 20 
```
- [ ] Limit max search run time in authorize.conf (role-level and/or default stanza) 
```
[role_foo]
SrchMaxTime = 2h
```

- [ ] Enable SSL and use signed certificates from a trusted root CA for Splunk Web 
	- https://help.splunk.com/en/splunk-enterprise/administer/manage-users-and-security/9.0/secure-splunk-platform-communications-with-transport-layer-security-certificates/configure-splunk-web-to-use-tls-certificates  
- [ ] Limit number of users with the admin role 
- [ ] Use Single Sign-On (SSO) for web access (if available) 
- Consider MFA as well https://docs.splunk.com/Documentation/Splunk/latest/Security/AboutMultiFactorAuth 
- [ ] Ensure that insecure encryption algorithms are disabled (SSL3, TLS 1.0, TLS 1.1) 
	- Applicable only to below Splunk v7.0.0 https://www.duanewaddle.com/quick-hit-disabling-sslv3-in-splunk/ 
- [ ] Use token-based authentication for REST API access 
- [ ] Configure reasonable search concurrency and disk quotas in splunk roles

## Search Head Clusters
- [ ] Configure pass4symmkey for shcluster 
- [ ] Lock-down SHC replication ports to only SHC members 
- [ ] Forward logs from your load balancer or reverse proxy 
	- Splunk Add-on for HAProxy 
	- Splunk Add-on for NGINX 
	- More on Splunkbase

### Important Links
- https://docs.splunk.com/Documentation/Splunk/latest/InheritedDeployment/Ports
- https://docs.splunk.com/Documentation/Splunk/latest/Security
- https://docs.splunk.com/Documentation/Splunk/latest/Security/AboutMultiFactorAuth
- https://docs.splunk.com/Documentation/Splunk/latest/Security/Setupauthenticationwithtokens
- https://docs.splunk.com/Documentation/Splunk/latest/DistSearch/Forwardsearchheaddata
- https://www.duanewaddle.com/quick-hit-disabling-sslv3-in-splunk/
- https://docs.splunk.com/Documentation/Splunk/latest/Security/Dataintegritycontrol#Configure_data_integrity_control
- https://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/Protectagainstthelossofin-flightdata
- https://docs.splunk.com/Documentation/Splunk/latest/Data/AboutHECIDXAck
- https://docs.splunk.com/Documentation/Splunk/latest/Data/UseHECusingconffiles#Global_settings
- https://wiki.splunk.com/images/f/fb/SplunkTrustApril-SSLipperySlopeRevisited.pdf
- https://www.duanewaddle.com/splunk-pass4symmkey-for-deployment-client-deployment-server/
- https://github.com/splunk/docker-swarm-splunk-hf
- https://downloads.cisecurity.org/download-issues/benchmarks
- https://galaxy.ansible.com/search?deprecated=false&keywords=cis%20benchmarks&order_by=-relevance&page=1
- https://opensource.com/article/18/3/ansible-patch-systems
