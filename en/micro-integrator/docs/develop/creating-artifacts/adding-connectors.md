# Adding Connectors

You can develop configurations with connectors, and deploy the configurations and connectors as composite application archive (CAR) files in WSO2 Micro Integrator using WSO2 Integration Studio.

!!! Info
    In addition to the below methods, you can enable a connector by creating a configuration file in the `MI_HOME/repository/deployment/server/synapse-configs/default/imports` directory with the following configurations.Replace the value of the `name` property with the name of your connector, and name the configuration file `{org.wso2.carbon.connector}<CONNECTOR_NAME>.xml` (e.g., `{org.wso2.carbon.connector}salesforce.xml`).
    ```
    <import xmlns="http://ws.apache.org/ns/synapse"
            name="salesforce"
            package="org.wso2.carbon.connector"
            status="enabled"/>
    ```

## Instructions

See the topics given below.

### Importing Connectors

Follow the steps below to import connectors into WSO2 Integration Studio:

1.  If you have already created an [ESB Config project](../../creating-projects/#esb-config-project), right click the ESB Config project where you want to use the connector and click **Add or Remove Connector**.
2.  On the wizard that appears, select **Add Connector** and click **Next**.
    -   If you have not downloaded any connectors, select the **Connector Store location** option to connect to the [connector store](https://store.wso2.com/store/) from WSO2 Integration Studio and import the required connectors into the workspace. Set `https://store.wso2.com:9448/` as the connector store location and click **Connect**. Select the required connectors and click **Finish**.
    -   If you have already downloaded the connectors, select the **Connector location** option and browse to the connector file from the file system. Click **Finish**. The connector is imported into the workspace and available for use with all the projects in the workspace.  
3.  After importing the connectors into WSO2 Integration Studio, the connector operations are available in the tool palette. You can drag and drop connector operations into your sequences and proxy services.

### Packaging Connectors

Follow the steps below to create a composite application archive (CAR) file containing the connectors:

1.  Click **File > New > Other** and select **Connector Exporter
    Project** under **WSO2 > Extensions > Project Types** and click
    **Next**. 
2.  Enter a project name and click **Finish**.  
3.  Right-click on the created connector exporter project, point to
    **New** and then click **Add/Remove Connectors**.
4.  Click **Add Connector** and then select **Workspace** . This will
    list down the connectors that have been imported into WSO2
    Integration Studio.
5.  Select the connector and click **OK**.
6.  Create a Composite Application (C-App) project including the
    required artifacts.
7.  Right-click on the C-App project and click **Export Composite
    Application Project** to create a CAR file out of that project.

### Removing Connectors

Follow the steps below to remove connectors from WSO2 Integration
Studio:

1.  Right-click on the relevant ESB Config project and click **Add or Remove Connector**.
2.  On the wizard that appears, select **Remove Connector** and click **Next**.
3.  Select the connectors you want to remove and click **Finish**.

## Examples
...

## Guides
