<img src="/img/SSL.jpg">

### I. Create a new user to automatically renew the certificate
- Control Panel -> User -> Create -> Enter a name and password (e.g: Name = "certadmin")
- Add user to "administrators" group.
- Deny access to all shared folders and applications (No access - Deny).
- Done.

### II. Create a ZeroSSL certificate for your domain

#### 1. Generate an access token for the ACME DNS API
- [Google Domain](https://domains.google.com/registrar/) -> example.com -> Security
- ACME DNS API -> Create token
- Copy the API token (e.g: UWRMY1JXOUp0cjBlYmhibmtJZm05dw==). Note that, it is only displayed once.

#### 2. Enable SSH
- Control Panel -> Terminal & SNMP -> Enable SSH service
- Login via SSH with your newly created admin user.

#### 3. Request a ZeroSSL certificate
- Download acme.sh to /usr/local/share/acme.sh/
```
wget -O /tmp/acme.sh.zip https://github.com/acmesh-official/acme.sh/archive/refs/heads/master.zip
sudo 7z x -o/usr/local/share /tmp/acme.sh.zip
sudo mv /usr/local/share/acme.sh-master/ /usr/local/share/acme.sh
sudo chown -R certadmin /usr/local/share/acme.sh/
```

- Set environment variables for [Google Domain](https://github.com/acmesh-official/acme.sh/wiki/dnsapi2#157-use-google-domains-dns-api) and ACME DNS API:
```
sudo -i
cd /usr/local/share/acme.sh
./acme.sh --install --force -m my@example.com
```
```
export GOOGLEDOMAINS_ACCESS_TOKEN="generated-access-token"
./acme.sh --issue --dns dns_googledomains -d *.example.com
```

- Set deployment options for [Synology DSM](https://github.com/acmesh-official/acme.sh/wiki/deployhooks#20-deploy-the-cert-into-synology-dsm):
```
export SYNO_Username="certadmin"
export SYNO_Password="P@ssw0rd"
export SYNO_Certificate="*.example.com"
export SYNO_Create=1
```

- Deploy the certificate into Synology DSM:
```
./acme.sh -d "*.example.com" --deploy --deploy-hook synology_dsm
```

- Go to Control Panel -> Security -> Certificate and set the new certificate as default.

### III. Setup a recurring task for renewal
- Control Panel -> Task Scheduler
- Create -> Scheduled Task -> User-defined script
- General: Give the task a name, set User = "certadmin" and check "Enabled"
- Schedule: Run on the following date -> Repeat monthly
- Task Settings: Insert the following scripts to "User-defined script":
```
/usr/local/share/acme.sh/acme.sh --cron --home /usr/local/share/acme.sh
```
