# DataStage Remote Engine using Docker

## License
### Cloud
[IBM DataStage as a Service Anywhere](https://www.ibm.com/support/customer/csol/terms/?ref=i126-9243-06-11-2023-zz-en)
### CP4D
[IBM DataStage Enterprise and IBM DataStage Enterprise Plus](https://www14.software.ibm.com/cgi-bin/weblap/lap.pl?li_formnum=L-QFYS-RTJPJH)

## Limitations
1. The script might not work on Apple `M` processors as it runs x86 software via an emulation layer.
2. The script is not tested on Windows OS.

## Pre-Requisites
1. If you are specifically deploying a remote engine for IBM Cloud, you must have DataStage provisioned on IBM Cloud with an `Anywhere` plan. You can see different plans with details on https://cloud.ibm.com/catalog/services/datastage.
1. Software that must be installed on the system.
    1. `docker` or `podman`
    1. `jq`
    1. `git` (optional)
1. You must have atleast 50GB of free space in `/var` in order to deploy the engine container. 200GB of free space is recommended.
1. You must have at least 2 cores and 4 GB memory, recommended is 4 cores and 8 GB memory, or more.
1. Recommended OS: Red Hat Enterprise Linux (RHEL 8.8, 8.10, 9.2 and 9.4), Ubuntu (20.04, 22.04, 24.04).
1. Ensure that the virtual machine allows outbound traffic to the following URLs:
    1. icr.io
    1. If you are specifically deploying a remote engine for IBM Cloud:
        1. iam.cloud.ibm.com
        1. dataplatform.cloud.ibm.com and api.dataplatform.cloud.ibm.com - if using Dallas datacenter
        1. eu-de.dataplatform.cloud.ibm.com and api.dataplatform.cloud.ibm.com - if using the Frankfurt data center
        1. au-syd.dai.cloud.ibm.com and api.au-syd.dai.cloud.ibm.com - if using the Sydney data center
        1. ca-tor.dai.cloud.ibm.com and api.ca-tor.dai.cloud.ibm.com - if using the Toronto data center
        1. cloud-object-storage.appdomain.cloud (the url could have a prefixed region eg. <region>.s3.cloud-object-store.appdomain.cloud), so recommendation is to allow `*.cloud-object-storage.appdomain.cloud` to accomodate such variations.


## Requirements
1. Clone this repo on the Remote Engine server: `git clone https://github.com/IBM/DataStage.git`.
    1. If you already have this repo cloned, go to the root directory and run `git pull` to get the latest changes.
1. If you are specifically deploying a remote engine for IBM Cloud, specify your IBM Cloud API Key. The API key is required for registering the remote engine to your Cloud Pak for Data project on IBM Cloud. To generate a new API key (https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui), open https://cloud.ibm.com in your browser.
    1. Click Manage > Access (IAM). Then, on the left side menu, select "API Keys" to open the "API Keys" page (URL: https://cloud.ibm.com/iam/apikeys).
    2. Ensure that My IBM Cloud API keys is selected in the View list.
    3. Click Create an IBM Cloud API key, and then specify a name and description.
       * Note: Once you close this pop-up window, you will not be able to access the value of this API Key again.
1. Your Encryption Key and IV. These can be generated by executing the command
    ```bash
    openssl enc -aes-256-cbc -k secret -P -md sha1
    ```
    Sample output:
    ```
    $ openssl enc -aes-256-cbc -k secret -P -md sha1
    salt=5334474DF6ECB3CC
    key=2A928E95489FCC163D46872040B9B24DC44E28A734B7681C8A3F0168F23E2A13
    iv =45990395FEB2B39C34B51D998E0E2E1B
    ```
    From this output, the `key` and `iv` are used as the Encryption Key and initialization vector (IV) respectively.
1. Your Project ID(s). This is the comma separated list of IDs of the projects on Cloud Pak for Data project that you want to use with this Remote Engine instance whether it is for IBM Cloud or CP4D. (If you are using the DataStage in Frankfurt data center then please use https://eu-de.dataplatform.cloud.ibm.com. If you are using the DataStage in Sydney data center then please use https://au-syd.dai.cloud.ibm.com. If you are using the DataStage in Toronto data center then please use https://ca-tor.dai.cloud.ibm.com.). You can retrieve this value by opening the project that you want to use with this Remote Engine and selecting the Manage tab > General to view the Project ID.
1. If you are specifically deploying a remote engine for IBM Cloud, the IBM Cloud Container Registry API Key. This API key will be used to download the images needed to run Remote Engine for IBM Cloud. Currently, clients need to request this API Key once they have provisioned a DataStage-aaS Plan and it needs to be requested via IBM Cloud Support: https://cloud.ibm.com/unifiedsupport.
1. If you are specifically deploying a remote engine for CP4D, the IBM Entitlement API key. This API key will be used to download the images needed to run Remote Engine for CP4D. Please follow https://www.ibm.com/docs/en/cloud-paks/cp-data/5.0.x?topic=information-obtaining-your-entitlement-api-key for instructions on how to obtain your IBM Entitlement API Key.

## Usage

### 1a. Start an engine on the Remote Engine server for IBM Cloud
The `dsengine.sh` script can be invoked from the `docker` folder of this project. Note that the name in the below command can be changed from `my_remote_engine_01` to your preferred name.
```bash
# create/start a local remote engine instance for IBM Cloud
./dsengine.sh start -n 'my_remote_engine_01' \
                    -a "$IBMCLOUD_APIKEY" \
                    -e "$ENCRYPTION_KEY" \
                    -i "$ENCRYPTION_IV" \
                    -p "$IBMCLOUD_CONTAINER_REGISTRY_APIKEY" \
                    --project-id "$PROJECT_ID1,$PROJECT_ID2,$PROJECT_ID3,..."

```
Once the script execution has completed, this engine needs to be selected in the project settings by going to the project, navigating to `Manage` > `DataStage` and selecting the appropriate engine under the `Settings` tab > `Remote` environments.
### 1b. Start an engine on the Remote Engine server for Cloud Pak for Data Instances
The `dsengine.sh` script can be invoked from the `docker` folder of this project. Note that the name in the below command can be changed from `my_remote_engine_01` to your preferred name.
```bash
# create/start a local remote engine instance for CP4D instance
./dsengine.sh start -n 'my_remote_engine_01' \
                    -e "$ENCRYPTION_KEY" \
                    -i "$ENCRYPTION_IV" \
                    -p "$IBM_ENTITLED_REGISTRY_APIKEY" \
                    --project-id "$PROJECT_ID1,$PROJECT_ID2,$PROJECT_ID3,..." \
                    --home "cp4d" \
                    --zen-url "CP4D_ZEN_URL" \
                    --cp4d-user "CP4D_USERNAME" \
                    --cp4d-apikey "CP4D_API_KEY"

```
Once the script execution has completed, this engine needs to be selected in the project settings by going to the project, navigating to `Manage` > `DataStage` and selecting the appropriate engine under the `Settings` tab > `Remote` environments.

#### Optional start flags

While starting a remote engine, following optional flags can be used in addition to the ones shown above. These can be seen via the help flag on the start subcommand: `./dsengine.sh start --help`.

1. `--memory <value>`: Sets the maximum amount of memory the engine can use. The value takes a positive integer, followed by a suffix of m/M, g/G, to indicate megabytes or gigabytes. Default is `4G`.
1. `--cpus <value>`: Sets the maximum amount of cpu resources the engine can use. The value takes a positive number. Default is `2` cores.
1. `--pids-limit <value>`: Set the PID limit of the container (defaults to -1 for unlimited pids for the container)
1. `--volume-dir <value>`: Sets the directory to be used as the volume directory for persistent storage. Default location is `/tmp/docker/volumes`. The volume directory will be updated with the following top level file structure:

    ```
    <volume_dir>/scratch
    <volume_dir>/ds-storage
    <volume_dir>/<remote_engine_name>_runtime
    <volume_dir>/<remote_engine_name>_runtime/px-storage
    ```
    Once the remote engine is up and running, additional files and folders will be created inside the above folders as needed by the engine.
    If you are planning to create multiple engines on the same machine, then they should use different volume directories.
1. `--home <value>`: Sets the target IBM Cloud enviroment to either `ypprod` (Dallas data center - default), `frprod` (Frankfurt data center), `sydprod` (Sydney data center), or  `torprod` (Toronto data center). The project associated with this engine instance must be in same data center.
1. `--select-version`: Set to true if you want to choose a specific version of remote engine. By default, this flag is set to false and the latest version is used.
1. `--security-opt <value>`: Specify the security-opt to be used to run the container.
1. `--cap-drop <value>`: Specify the cap-drop to be used to run the container.
1. `--set-user <username>`: Specify the username to be used to run the container. If not set, the current user is used.
1. `--set-group <groupname>`: Specify the group to be used to run the container.
1. `--additional-users <IBMid-1000000000,IBMid-2000000000,IBMid-3000000000,...>`: Comma separated list of ids (IAM IDs for cloud, check https://cloud.ibm.com/docs/account?topic=account-identity-overview for details; uids/usernames for cp4d) that can also control remote engine besides the owner.
1. `--mount-dir "</path/to/local_dir:/path/on/container>"`: Specify folder you want to mount on the container. This flag can be specified multiple times.
    * Note: This flag can be used to mount a scratch directory using `--mount-dir "</path/to/local_dir>:/opt/ibm/PXService/Server/scratch"`. This will override the default scratch directory that is either created in  `/tmp` or in the directory specified for `--volume-dir`.
1. `--add-host <host>:<ip>`: Add a <host>:<ip> entry to the /etc/hosts file of the container. This flag can be specified multiple times.
1. `--proxy http://<username>:<password>@<proxy_ip>:<port>`: Specify a proxy URL. The username and password can be skipped based on how the proxy is configured.
1. `--proxy-cacert <cacert location>`: Specify the location of the custom CA store for the specified proxy - if it is using a self signed certificate.
1. `--force-renew`: Set to true if you want to remove the existing remote engine container. By default, this flag is set to false and if a stopped existing container is found, it is restarted or if a running existing container is found, the script is aborted.
1. `--zen-url`: CP4D zen url of the cluster (required if --home is used with "cp4d").
1. `--cp4d-user`: CP4D username used to log into the cluster (required if --home is used with "cp4d").
1. `--cp4d-apikey`: CP4D apikey used to authenticate with the cluster. Go to "Profile and settings" when logged in to get your api key for the connection. (required if --home is used with "cp4d").
1. `--registry`: Custom container registry to pull images from. Must also set -u and -p options to login to the registry as well as either --digest or --image-tag for IBM Cloud.
1. `-u | --user`: User to login to a custom container registry (required if --registry is set).
1. `--digest`: Digest to pull the ds-px-runtime image from the registry (required if --registry is set and --image-tag is not set).
1. `--image-tag`: Image tag to pull the ds-px-runtime image from the registry (required if --registry is set and --digest is not set).
1. `--skip-docker-login`: [true | false]. Skips Docker login to container registry if that step is not needed.
1. `--env-vars`: Semi-colon separated list of key=value pairs of environment variables to set (eg. key1=value1;key2=value2;key3=value3;...). Whitespaces are ignored.
    * Remote Engine specific environment variables:
        * REMOTE_ENGINE_BATCH_SIZE - Set to an integer representing the maximum number of jobs that remote engine will pull at one time. Default value is 5.
        * APT_USE_REMOTE_APP - Set to "force" to make remote engine avoid forking section leader processes. Can avoid inheriting unwanted open resources from the conductor. Default is unset.
        * ENABLE_DS_METRICS - Set to "true" to have the remote engine send metrics to a configured DataStage metrics repository. See the [IBM Cloud](https://dataplatform.cloud.ibm.com/docs/content/dstage/dsnav/topics/ds_metrics.html?context=cpdaas&audience=wdp) or [Cloud Pak for Data](https://www.ibm.com/docs/en/software-hub/5.1.x?topic=administering-storing-persisting-metrics) documentation for more information.


### 2. Update an engine

Update the remote engine instance to a new version. The update command gathers information from the existing docker container, then stops the existing container, removes it and spins up a new container with the same name. Hence a container must be running prior to running the the update command. You can simply re-run the start command that you started the container with.

```bash
# this will update the container with name 'my_remote_engine_01'
./dsengine.sh update -n "my_remote_engine_01" \
                     -p "$IBMCLOUD_CONTAINER_REGISTRY_APIKEY"
```

#### Optional update flags

While updating a remote engine, following optional flags can be used in addition to the ones shown above. These can be seen via the help flag on the start subcommand: `./dsengine.sh update --help`.

1. `--select-version`: Set to true if you want to choose a specific version of remote engine. By default, this flag is set to false and the latest version is used.
1. `--proxy http://<username>:<password>@<proxy_ip>:<port>`: Specify a proxy URL. The username and password can be skipped based on how the proxy is configured.


### 3. Stop an engine

Stop the remote engine container.
```bash
# stop a local remote engine instance with name 'my_remote_engine_01'
./dsengine.sh stop -n 'my_remote_engine_01'

```
Note that if the `./dsengine.sh start` is used when a container is stopped, the script will simply start the stopped container.

### 4. Cleanup/Uninstall a remote engine
This is NOT needed use this if you want to update the engine. This is only needed if you want to completely remove the engine container, delete the volume directories and deregister the remote engine from the associated project.
```bash
# cleanup a remote engine instance with name 'my_remote_engine_01'
./dsengine.sh cleanup -n 'my_remote_engine_01' \
                      -a "$IBMCLOUD_APIKEY" \
                      --project-id "$PROJECT_ID"
```

### Troubleshooting

1. Flow execution fails with incorrect host or IP Address in the log
If the script finishes succesfully, and you are able to see the engine in your project but the run still fails, you can check if container hosts file contains the host IP address. If so, you can remove the host IP address mapping so that the container uses 127.0.0.1 to resolve the hostname. This issue has been seen primarily when using podman. You can edit this file using the below steps
    1. Find the running container name or id using
       ```bash
       docker ps
       # OR
       podman ps
       ```
    1. Exec into the container:
       ```bash
       docker exec -it <container-name-or-id> bash
       # OR
       podman exec -it <container-name-or-id> bash
       ```
    1. Edit the hosts file inside the container
       ```bash
       nano /etc/hosts
       ```
    1. Remove the line that contains the IP address of the host and the host name
    1. Press `ctrl + x` to save and press `y` to exit from nano.
    1. Retry the flow.
Note that this fix will need to be re-applied whenever the current container is removed, eg. updating to a new image.

2. Making sure the URLs are allowlisted
For URLs mentioned in the [pre-requisites](https://github.com/IBM/DataStage/blob/main/RemoteEngine/docker/README.md#pre-requisites) section above, you can use ping from the host vm to make sure the URLs are accessible. Eg.
    ```
    $ ping api.dataplatform.cloud.ibm.com
    PING api.dataplatform.cloud.ibm.com (172.66.129.176) 56(84) bytes of data.
    64 bytes from 172.66.129.176 (172.66.129.176): icmp_seq=1 ttl=53 time=6.66 ms
    64 bytes from 172.66.129.176 (172.66.129.176): icmp_seq=2 ttl=53 time=6.90 ms
    64 bytes from 172.66.129.176 (172.66.129.176): icmp_seq=3 ttl=53 time=6.82 ms
    64 bytes from 172.66.129.176 (172.66.129.176): icmp_seq=4 ttl=53 time=6.79 ms
    64 bytes from 172.66.129.176 (172.66.129.176): icmp_seq=5 ttl=53 time=6.87 ms
    ^C
    --- api.dataplatform.cloud.ibm.com ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4007ms
    rtt min/avg/max/mdev = 6.662/6.806/6.898/0.081 ms
    ```
