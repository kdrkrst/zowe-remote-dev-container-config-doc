# Using Zowe Explorer with Remote Dev Containers
This repository is containing a sample devcontainer.json configuration file to help developers using VS Code Extension for Zowe in remote dev containers.
## What is VS Code Extension for Zowe?
The Zowe Explorer extension for Visual Studio Code (VS Code) modernizes the way developers and system administrators interact with z/OS mainframes, and lets you interact with data sets, USS files, and jobs. URL to the extension: https://docs.zowe.org/stable/user-guide/ze-install/
## What is Remote Dev Containers?
Remote Dev Containers allow developers to work inside a container on a remote server. You can use the Remote - SSH or Remote - Tunnels extensions to connect to a remote host, create and configure a dev container, and develop code within it. Additionally, you can use Docker contexts or specialized settings to attach to containers on a remote Docker host. These approaches enable efficient development without needing to mount local files into the container.  
Remote dev containers allow developers to work in a consistent, isolated environment. This ensures that dependencies, configurations, and tools are uniform across team members, reducing compatibility issues and making collaboration smoother.  
For example, a dev container can provide a convenient way to develop an application requiring Java 22, Node.js, npm, and specific Linux libraries like libsecret-1-0. The isolation benefits of dev containers ensure that each project has its own environment, avoiding conflicts between dependencies of other applications.  
![image](https://github.com/kdrkrst/zowe-remote-dev-container-config-doc/assets/1232685/2e976653-c0bb-4f9a-be73-ca8a9eeb04f1)
## Remote Dev Container Configuration for VS Code Extension for Zowe
The VS Code Extension for Zowe requires specific configurations within the development container:
* Installion of node.js
*	Installation of Java SDK
  
This can be configured by setting up the `features` section in the `devcontainer.json` file as follows:
```json
"features": {
  "ghcr.io/devcontainers/features/java:1": {
    "version": "none",
    "installMaven": "true",
    "installGradle": "false"
  },
  "ghcr.io/devcontainers/features/node:1": {
    "nodeGypDependencies": true,
    "version": "latest",
    "nvmVersion": "latest"
  }
}
```
However, this configuration alone may lead to certain errors in Visual Studio Code, such as:
> Zowe Explorer failed to activate since the default credential manager is not supported in your environment.
> Do you wish to disable credential management? (VS Code window reload will be triggered)
> ![image](https://github.com/kdrkrst/zowe-remote-dev-container-config-doc/assets/1232685/8f7de840-7305-4e0b-a44e-3351d7316dce)
  
In the error message displayed, none of the buttons will resolve the issue with the VS Code for Zowe Extension. You are likely to see these errors:
> IBM Z Open Editor was not able to find a valid Zowe CLI profile to use. Review the following error message and try to fix the problem by checking if valid entries were defined for zopeneditor.zowe in your VS Code user settings: "Failed to initialize secure credential manager"
  
> Error: Unable to initialize secure credentials plugin for Z Open Editor. Check that you configured the "Zowe Security: Credential Key" user preferences with the value "Zowe-Plugin". Error returned: "Failed to initialize secure credential manager"

> Error: Failed to load credential manager. This may be related to Zowe Explorer being unable to use the default credential manager in a browser based environment. Instead, you should activate the necessary packages within the container by appending the following segment to the `features` section.

![image](https://github.com/kdrkrst/zowe-remote-dev-container-config-doc/assets/1232685/94eb1012-e042-450d-9a61-32ba3509be59)

These issues arise from the absence of specific packages in the container. To activate these packages, add the following section to the `features` section.

```json
"ghcr.io/devcontainers-contrib/features/apt-get-packages:1": {
  "packages": "libsecret-1-0,dbus-x11,gnome-keyring"
}
```
This time, the error will transform into something similar to the following:
> No machine-id detected in docker containers
  
The issue requires making the host machine's ID visible inside the container. You can achieve this adding following mounts in the `devcontainer.json` file:
```json
"mounts": [
  "source=/etc/machine-id,target=/etc/machine-id,type=bind,consistency=cached",
  "source=/var/lib/dbus/machine-id,target=/var/lib/dbus/machine-id,type=bind,consistency=cached"
]
```
The last error that needs to be addressed is:
> /usr/bin/gnome-keyring-daemon: Operation not permitted
  
This is the most complex issue since there is no indication as to why the operation is not permitted. Bizarrely, this error is raised because the gnome-keyring-daemon requires a very specific capability to be present in the container: `IPC_LOCK`.  
To solve this issue, you can add this section to the `devcontainer.json` file:
```json
"capAdd": [
    "IPC_LOCK"
]
```
# Final Content of the Sample devcontainer.json
```js// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/java
{
	"name": "Java",
	// Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
	"image": "mcr.microsoft.com/devcontainers/java:1-21-bullseye",

	"features": {
		"ghcr.io/devcontainers/features/java:1": {
			"version": "none",
			"installMaven": "true",
			"installGradle": "false"
		},
		"ghcr.io/devcontainers/features/node:1": {
			"nodeGypDependencies": true,
			"version": "latest",
			"nvmVersion": "latest"
		},
		"ghcr.io/devcontainers-contrib/features/apt-get-packages:1": {
			"packages": "libsecret-1-0,dbus-x11,gnome-keyring"
		}
	},

	"mounts": [
		"source=/etc/machine-id,target=/etc/machine-id,type=bind,consistency=cached",
		"source=/var/lib/dbus/machine-id,target=/var/lib/dbus/machine-id,type=bind,consistency=cached"
	],

	"capAdd": [
	    "IPC_LOCK"
	  ]

	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	// "forwardPorts": [],

	// Use 'postCreateCommand' to run commands after the container is created.
	// "postCreateCommand": "java -version",

	// Configure tool-specific properties.
	// "customizations": {},

	// Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.
	// "remoteUser": "root"
}
```
