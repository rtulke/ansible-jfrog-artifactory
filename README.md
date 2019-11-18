# ansible-jfrog-artifactory
Download Images from via jfrog from artifactory

## requirements

* JFROG Installation and User specific configuration see at jfrog_installation.md
* mkdir -p /opt/templates/

## install playbook

```
git clone https://github.com/rtulke/ansible-jfrog-artifactory/
```

change to ansible playbook folder

```
cd ansible-jfrog-artifactory/
```

## examples

### download_image_via_jfrog.yml

```
ansible-playbook download_image_via_jfrog.yml -e "artifact=templates-generic-local artifact_url_env=prd artifact_url_release=20191101 artifact_url_image=RHEL7-U7-vmware-custom-20191101"
```

Check if the download was successful.

```
find /opt/templates/artifacts/
/opt/templates/artifacts/
/opt/templates/artifacts/prd
/opt/templates/artifacts/prd/20191101
/opt/templates/artifacts/prd/20191101/RHEL7-U7-vmware-custom-managed-20191101
/opt/templates/artifacts/prd/20191101/RHEL7-U7-vmware-custom-managed-20191101/RHEL7-U7-vmware-custom-managed-20191101.ova
...
...
```

#### variables and parameters

These variables can be passed to the Playbook via parameters.

```
artifact:               "templates-generic-local"
artifact_url_env:       "prd"
artifact_url_release:   "20191101"
artifact_url_image:     "RHEL7-U7-vmware-custom-20191101"
```

These more system specific variables can also be passed to the Playbook via parameters. But should not be necessary, because these variables are set as default by vars.yml

```
jfrog_env_log_level:    "DEBUG" # Possible values are: INFO, ERROR, and DEBUG
jfrog_env_temp_dir:     "/opt/templates/tmp/"
jfrog_local_target_dir: "/opt/templates/artifacts"
```

## Troubleshooting

The download does not work but Ansible does not show an error. There can be several reasons for this problem.

* Check the local environment settings

Normally Ansible should set the environment variables itself during a play-run but you can also set them directly.

```
env |grep JFROG

JFROG_CLI_TEMP_DIR=/opt/templates/tmp/
JFROG_CLI_LOG_LEVEL=DEBUG
```

try to export it manually

```
export JFROG_CLI_TEMP_DIR="/opt/templates/tmp/"
export JFROG_CLI_LOG_LEVEL="DEBUG"
```

or save it in the ~/.bash_profile don't forget to source ~/.bash_profile or reconnect the session after adding...

```
source ~/.bash_profile
```

* Check if the file ~/.jfrog/jfrog-cli.conf was saved in a clean JSON format. Check the syntax.

```
cat ~/.jfrog/jfrog-cli.conf

{
  "artifactory": [
    {
      "url": "https://artifactory.myhost.com/",
      "serverId": "false",
      "apiKey": "YOUR-TOKEN-HERE",
      "isDefault": true
    }
  ],
  "Version": "1"
}
```


You can verify JSON syntax with jsonlint or try http://zaa.ch/jsonlint/ (but remove token and the url!)

* Check if the token defined in the ~/.jfrog/jfrog-cli.conf file is current and matches the one in the artifactory.

```
curl -H 'X-JFrog-Art-Api:YOUR-TOKEN-HERE' -O "https://artifactory.myhost.com/templates-generic-local/prd/20191001/RHEL7-U7-vmware-custom-20191101/RHEL7-U7-vmware-custom-20191101.ovf"
```

* Check Proxy Settings if needed

```
env |grep proxy

http_proxy=proxy.server.host.com:8080
https_proxy=proxy.server.host.com:8080
```

* Test the Connectivity to the Proxy Server, if required
 
```
if nc -z -v -w5 proxy.server.host.com  443; then echo "connection OK"; else echo "connection not OK"; fi

Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 1xx.2xx.3xx.4xx:443.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
```

* Check the Connectivity to the artifacotry test with netcat

```
if nc -z -v -w5 artifactory.myhost.com 443; then echo "connection OK"; else echo "connection not OK"; fi

Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 1xx.2xx.3xx.4xx:443.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
connection OK
```

* Check user rights 

```
find /opt/templates/ -type f -print0 | xargs -0 ls -l
```

The directories /opt/templates/prd/, /opt/templates/tmp/ and /opt/templates/artifacts/ needs ansible group rights.

```
sudo chown -R root:ansible /opt/templates/tmp/
sudo chown -R root:ansible /opt/templates/artifacts/
```

```
sudo chmod 775 /opt/templates/tmp/
sudo chmod 775 /opt/templates/artifacts/

```

```
ls -lat /opt/templates/
```

```
total 0
drwxr-xr-x. 4 root root     34 Nov 15 14:03 .
drwxrwxr-x. 6 root ansible 114 Nov 15 14:01 tmp
drwxrwxr-x. 4 root ansible  28 Nov 15 13:53 artifacts
drwxr-xr-x. 6 root root     70 Sep 12 16:30 ..

```
