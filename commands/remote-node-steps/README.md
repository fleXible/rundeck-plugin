Use *remote-node-steps* to generate a plugin that will let users execute
the module commands as [RemoteScriptNodeStep Plugins](http://rundeck.org/docs/developer/workflow-step-plugin.html#remotescriptnodestep-plugin).
With your modules contained in the plugin, Rundeck will take care of
deploying the rerun modules to the remote host (as a rerun archive)
and execute the user's specified command.

Job writers will enjoy using your rerun modules as a plugins because
they can use the Rundeck GUI or text formats to define Job workflows.

You can specify a list of modules using a glob pattern. Every command
contained in your modules becomes a node step you can reference in
your job steps.

Imagine you have a module called "waitfor" that has a command called "ping" you'd
like to make accessible as a Job step.

The usage is as described below:

	$ rerun waitfor
	Available commands in module, "waitfor":
	ping: "wait for ping response from host"
	    --host <"">: "the host to reach"
	   [ --interval <30>]: "seconds between checks"

Create a plugin for it:

    rerun rundeck-plugin:remote-node-steps --modules waitfor --name waitfor

The `--modules` option specifies which list of modules to include.
The `--name` option gives your plugin a name.

Each command in the module will be exported as a plugin service named after the command using the _MODULE:COMMAND_ format (eg, waitfor:ping).

### Deploying the plugin

Copy the plugin archive to your rundeck:

    cp ./build/waitfor.zip $RDECK_BASE/libext

You should immediately be able to use the plugin.

### Defining Jobs using your step plugin

Once your module has been generated into a plugin you can call its commands as Job steps.

Listing: my-job.yaml

	- id: 7d53b629-2c9c-4c66-8585-87da82e3d668
	  name: 'my job'
	  description: 'a job that shows how to call a command in my plugin'  
	  loglevel: INFO
	  sequence:
	    keepgoing: false
	    strategy: node-first
	    commands:
	    - type: waitfor:ping
	      nodeStep: true
	      configuration:
	        host: google.com
	        interval: 30
	      description: this is a step description

You can see that the command 'type' is the MODULE:COMMAND (e.g., waitfor:ping).

Command options are expressed as 'configuration' properties (e.g., host and interval).

See [Document Format Reference / JOB-YAML](http://rundeck.org/docs/man5/job-yaml.html#plugin-step-entry)
for more about defining jobs in YAML files.

## SETUP

The plugin passes environment variables to the remote nodes.

Ensure `RD_*` environment variables are passed to remote nodes by configuring your sshd correctly.
See the [Rundeck Aministration Guide](http://rundeck.org/docs/plugins-user-guide/ssh-plugins.html#passing-environment-variables-through-remote-command)

## Project configuration

There might be cases where you want to hide a command option from the
job writer and runner. 
Instead you will make it configurable by the Rundeck adminsitrator.

Add the `RUNDECK_PLUGIN_CONFIG_SCOPE` property to the metadata file for the option
you want to make a project configuration variable (eg, `RUNDECK_PLUGIN_CONFIG_SCOPE=Project`):

	# option metadata
	# generated by stubbs:add-option
	# Wed Jul  8 18:18:38 PDT 2015
	NAME=api-key
	DESCRIPTION="the API Key used to access the server"
	ARGUMENTS=true
	REQUIRED=true
	SHORT=
	LONG=api-key
	DEFAULT=
	EXPORT=false
	RUNDECK_PLUGIN_CONFIG_SCOPE=Project
	RUNDECK_PLUGIN_CONFIG_RENDERINGOPTIONS_DISPLAYTYPE=PASSWORD

Your plugin will be built in such a way that the api-key is not presented
to the job writer as a plugin parameter.

If you are creating a new project via the CLI, you can configure the plugin like so:

	rd-project -a create -p myproject \
	    --project.plugin.RemoteScriptNodeStep.waitfor.api-key=3D5D004C-A9BC-476E-9162-6E176ADFF914

