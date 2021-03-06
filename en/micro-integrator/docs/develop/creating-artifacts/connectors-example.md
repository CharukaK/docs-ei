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

## Importing connectors

Follow the steps below to import connectors into WSO2 Integration
Studio:

1.  Right-click on the ESB Config project where you want to use the connector and click **Add or Remove Connector** .  
    ![](attachments/119131934/119131946.png)
2.  On the wizard that appears, select **Add Connector** and click **Next**.
    -   If you have not downloaded any connectors, select the **Connector Store location** option to connect to the [connector
        store](https://store.wso2.com/store/) from WSO2 Integration Studio and import the required connectors into the workspace. Set `https://store.wso2.com:9448/` as the connector store location and click **Connect**. Select the required connectors and click **Finish**.  
        ![](attachments/119131934/119131945.png)
    -   If you have already downloaded the connectors, select the **Connector location** option and browse to the connector file from the 
        file system. Click **Finish** . The connector is imported into the workspace and available for use with all the projects in the workspace.  
        ![](attachments/119131934/119131944.png)
3.  After importing the connectors into WSO2 Integration Studio, the
    connector operations are available in the tool palette. You can drag
    and drop connector operations into your sequences and proxy
    services.  
    ![](attachments/119131934/119131943.png){width="800" height="328"}

For complete information on each of the predefined connectors, see the [ESB connectors](https://docs.wso2.com/display/ESBCONNECTORS/WSO2+ESB+Connectors+Documentation).

## Creating a CAR file including connectors

Follow the steps below to create a composite application archive (CAR)
file containing the connectors:

1.  Click **File > New > Other** and select **Connector Exporter
    Project** under **WSO2 > Extensions > Project Types** and click
    **Next** .  
    ![](attachments/119131934/119131942.png)
2.  Enter a project name and click **Finish** .  
    ![](attachments/119131934/119131941.png)
3.  Right-click on the created connector exporter project, point to
    **New** and then click **Add/Remove Connectors** .
4.  ![](attachments/119131934/119131940.png)
5.  Click **Add Connector** and then select **Workspace** . This will
    list down the connectors that have been imported into WSO2
    Integration Studio.

    ![](attachments/119131934/119131939.png)

6.  Select the connector and click **OK** .
7.  Create a Composite Application (C-App) project including the
    required artifacts.  
    ![](attachments/119131934/119131937.png)
8.  Right-click on the C-App project and click **Export Composite
    Application Project** to create a CAR file out of that project.  
    ![](attachments/119131934/119131936.png)

## Removing connectors

Follow the steps below to remove connectors from WSO2 Integration
Studio:

1.  Right-click on the relevant ESB Config project and click **Add or
    Remove Connector** .
2.  On the wizard that appears, select **Remove Connector** and click
    **Next** .
3.  Select the connectors you want to remove and click **Finish** .  
    ![](attachments/119131934/119131935.png){width="400" height="439"}

## Example

When a connector is enabled in your WSO2 EI instance, you can access its
functionality by adding a reference to it from your configuration (such
as in a sequence) and then calling its operations. To see the operations
available in a connector, view your list of connectors (on the Main tab
of the Management Console, under **Connectors** click **List** ), and
then click the connector name.

For example, if the JIRA connector is enabled in your WSO2 EI instance,
and you want to use it in a sequence, you would add the following entry
to your sequence configuration to connect to JIRA:

```
<jira.init>
   <username>myname@mycompany.com</username>
   <password>{wso2:vault-lookup('xxxx.email.login')}</password>
   <uri>https://xxx:xx/xx</uri>
</jira.init>
```

Then, if you want to get an issue, you can use the JIRA connector's `getIssue` operation and pass in the issue ID as
follows:

```
<jira.getIssue [configKey="Ref Local Entry"]>
    <issueIdOrKey>EIJAVA-2095</issueIdOrKey>
</jira.getIssue>
```
