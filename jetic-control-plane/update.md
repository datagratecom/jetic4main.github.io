### Update `jetic-bridge` and `jetic-operator` to `jetic-control-plane`

1. Ensure all changes are committed and pushed.  
1.  To Backup jetic-bridge config, Run:  
    ```sh
    helm get values jetic-bridge -n namespace > values.yaml
    ```
1.  To Backup jetic-operator config, Run:  
    ```sh
    helm get values jetic-operator -n namespace > operatorValues.yaml
    ```
1.  Uninstall the old components:  
    ```sh
    helm uninstall jetic-bridge -n namespace
    helm uninstall jetic-operator -n namespace
    ```
1.  Merge `values.yaml` and `operatorValues.yaml` into `jetic.yaml`, adding any additional configuration required for the cluster/bridge setup from `values.yaml`:

    ```yaml
    cluster:
      key: <key>
      secret: <secret>
      ssl: true
    docker:
      address: docker.io
      registry: https://index.docker.io/v1/
      username: <username>
      token: <token>
    ```

1.  Download the required CRDs:  
    ```sh
    wget https://helm.jetic.io/jetic-control-plane/crds.yaml -O crds.yaml
    ```
    ```sh
    curl https://helm.jetic.io/jetic-control-plane/crds.yaml > crds.yaml
    ```
1.  Apply the CRDs:  
    ```sh
    kubectl replace -f crds.yaml
    ```
    > **Note:** In some cases, templates may be missing. If that happens, running `kubectl create replace` is a safe operation. **DO NOT execute a `DELETE` on the CRDs**, as that will interfere with running integrations and remove the ability to rebuild them.

1.  Add and update the Helm repository:  
    ```sh
    helm repo add jetic-control-plane https://helm.jetic.io/jetic-control-plane/charts/
    helm repo update
    ```

1.  Install the new Jetic Control Plane:  
    ```sh
    helm install jetic-control-plane-dev -f jetic.yaml -n namespace
    ```

1.  If necessary, clone your repository again and confirm that you can still access your integrations and retrieve logs.  

1.  Download the Camel-K CLI for your OS:  
    [https://downloads.apache.org/camel/camel-k/2.5.1/](https://downloads.apache.org/camel/camel-k/2.5.1/)  

1.  Extract the downloaded file and grant execution permissions to `kamel`:  
    ```sh
    chmod 775 kamel  # Linux/Unix
    ```

1.  Verify the installation:  
    ```sh
    ./kamel get -n namespace
    ```

1.  Rebuild an integration:  
    ```sh
    ./kamel rebuild <integration>
    ```
    > **Note:** Mapping integrations will fail if rebuilt and must be manually triggered from Jetic.  
    > This can be done using **Stop/Run** even without rebuilding first.

1.  Monitor integrations:  
    ```sh
    kubectl get integrations -n staging -w
    ```
    Once an integration is running, go back to step **14** and continue with the next integration.