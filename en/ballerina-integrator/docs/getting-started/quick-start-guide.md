# Quick Start Guide

This section helps you to quickly set up and run Ballerina so that you can use it for various solutions. Here we 

## Install Ballerina

1. [Download](https://ballerina.io/downloads) Ballerina for your Operating System. 
1. Follow the instructions given in the [Ballerina Getting Started page](https://ballerina.io/learn/getting-started/) to set it up. 

> **Note**: Throughout this documentation, `<ballerina_home>` refers to the directory in which you just installed Ballerina.

## Set up the Editor

Let's try this on VS Code.

> **Tip:** You can use your [favorite editor to work on Ballerina code](https://ballerina.io/learn/tools-ides/), however, we recommend you use VS Code as we have made some improvements to it for use in integration scenarios.

1. Open VS Code.
   > **Tip**: Download and install [VS Code](https://code.visualstudio.com/Download) if you do not have it already.

2. Find the extension for Ballerina in the VS Code marketplace. For instructions on installing and using it, see [The Visual Studio Code Extension](https://ballerina.io/learn/tools-ides/vscode-plugin/).

Once you have installed the extension, press `Command + Shift + P` in Mac or `Ctrl + Shift + P` in Linux and the following page appears.

![alt text](../../assets/img/vs-code-landing.png)

You can select one of the available templates or run it using the CLI as indicated in the following section.

## Create a Project, Add a Template, and Invoke the Service

Create a new project by navigating to a directory of your choice and running the following command. 

```bash
$ ballerina new MyProject
```

You see a response confirming that your project is created.

In this project, you will be creating a service that transforms an XML message to JSON.

![alt text](../../assets/img/xml-json.png)

First let's pull the module from Ballerina Central, which is a public directory that allows you to host templates and modules. In this case, we use the `xml_to_json_transformation` template.

```
$ ballerina pull wso2/xml_to_json_transformation
```

Navigate into the project directory you created and run the following command. This command enables you to create a module using the predefined template you pulled.

```bash
$ ballerina add -t wso2/xml_to_json_transformation MyModule
```

This automatically creates an XML to JSON transformation service for you inside an `src` directory. A Ballerina service represents a collection of network accessible entry points in Ballerina. A resource within a service represents one such entry point. The generated sample service exposes a network entry point on port 9090.

Build the service using the `ballerina build` command.

```bash
$ ballerina build MyModule
```

You get the following output.

```bash
Compiling source
	sam/MyModule:0.1.0

Creating balos
	target/balo/MyModule-2019r3-any-0.1.0.balo

Running tests
    sam/MyModule:0.1.0
	No tests found


Generating executables
	target/bin/MyModule.jar
```

Run the following Java command to run the executable .jar file that is created once you build your module.

```
java -jar target/bin/MyModule.jar --b7a.config.file=src/MyModule/resources/ballerina.conf
```

Your service is now up and running. You can invoke the service using an HTTP client. In this case, we use cURL.

> **Tip**: If you do not have cURL installed, you can download it from [https://curl.haxx.se/download.html](https://curl.haxx.se/download.html).

First we create an XML file inside `MyModule`.

```xml

<user>
    <name>Sam</name>
    <job>Scientist</job>
</user>

```

Invoke the service using the following curl command.

```bash
$ curl -X POST -d @request.xml  http://localhost:9092/laboratory/user  -H "Content-Type: text/xml"
```

You get the following response.

```xml
<response>
    <status>successful</status>
    <id>987</id>
    <name>Sam</name>
    <job>Scientist</job>
    <createdAt>2019-09-10T11:47:38.387Z</createdAt>
</response>

```

You just started Ballerina, created a project, started a service, invoked the service you created, and received a response.

To have a look at the code, navigate to the `main.bal` file found inside your module.

```
// Copyright (c) 2019 WSO2 Inc. (http://www.wso2.org) All Rights Reserved.
//
// WSO2 Inc. licenses this file to you under the Apache License,
// Version 2.0 (the "License"); you may not use this file except
// in compliance with the License.
// You may obtain a copy of the License at
//
// http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing,
// software distributed under the License is distributed on an
// "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
// KIND, either express or implied.  See the License for the
// specific language governing permissions and limitations
// under the License.

import ballerina/config;
import ballerina/http;
import ballerina/log;

// Endpoint for the backend service.
http:Client healthcareEndpoint = new("https://reqres.in");
// Constants for request paths.
const BACKEND_EP_PATH = "/api/users";

@http:ServiceConfig {
    basePath: "/laboratory"
}
service scienceLabService on new http:Listener(config:getAsInt("LISTENER_PORT")) {
    // Schedule an appointment.
    @http:ResourceConfig {
        methods: ["POST"],
        path: "/user"
    }
    resource function addUser(http:Caller caller, http:Request request) {
        // Extract payload from the request.
        xml|error requestPayload = request.getXmlPayload();

        if (requestPayload is xml) {
            // Transform the payload into the json which is required by the backend service.
            json|error modifiedPayload = {
                "name": requestPayload.name.getTextValue(),
                "job": requestPayload.job.getTextValue()
            };

            if (modifiedPayload is json) {
                // Create new request to call the back-end service with the modified payload.
                http:Request backendRequest = new();
                backendRequest.setPayload(<@untainted> modifiedPayload);
                // Define new response
                http:Response|error backendResponse = new();
                backendResponse = healthcareEndpoint->post(BACKEND_EP_PATH, backendRequest);

                if (backendResponse is http:Response) {
                    // Get json payload from the backend response.
                    json|error resJson = backendResponse.getJsonPayload();

                    if (resJson is json) {
                        // Convert the json payload to xml.
                        string id = resJson.id.toString();
                        string name = resJson.name.toString();
                        string job = resJson.job.toString();
                        string created = resJson.createdAt.toString();

                        xml|error resXml = xml `<response>
                            <status>successful</status>
                            <id>${id}</id>
                            <name>${name}</name>
                            <job>${job}</job>
                            <createdAt>${created}</createdAt>
                        </response>`;

                        if (resXml is xml) {
                            respondAndHandleError(caller, http:STATUS_OK, <@untainted> resXml);
                        } else {
                            logAndRespondError(caller, "Converting backend json response to xml failed.", resXml,
                                http:STATUS_INTERNAL_SERVER_ERROR);
                        }

                    } else {
                        logAndRespondError(caller, "Invalid response from backend service.", resJson,
                            http:STATUS_INTERNAL_SERVER_ERROR);
                    }

                } else {
                    logAndRespondError(caller, "Error in sending request to backend service.", backendResponse, 
                        http:STATUS_INTERNAL_SERVER_ERROR);
                }

            } else {
                logAndRespondError(caller, "Transforming xml to json failed.", modifiedPayload, 
                    http:STATUS_INTERNAL_SERVER_ERROR);
            }

        } else {
            logAndRespondError(caller, "Invalid request payload.", requestPayload, http:STATUS_BAD_REQUEST);
        }
    }
}

// Log and respond error.
function logAndRespondError(http:Caller caller, string errMsg, error err, int statusCode) {
    log:printError(errMsg, err = err);
    respondAndHandleError(caller, statusCode, errMsg);
}

// Send the response back to the client and handle responding errors.
function respondAndHandleError(http:Caller caller, int resCode, json|xml|string payload) {
    http:Response res = new;
    res.statusCode = resCode;
    res.setPayload(payload);
    var respond = caller->respond(res);
    if (respond is error) {
        log:printError("Error occurred while responding", err = respond);
    }
}
```
