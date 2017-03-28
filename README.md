# Docker Compose Wrapper

Docker Compose Wrapper is a poor-man PAAS management tool.
This script provides a wrapper to the *docker-compose* command and permits to expose commands that can be executed on the Docker host.

The common use-case for this tool is to be used as an SSH command executed trough the `~/.ssh/authorized_keys` file, see below.

### SECURITY CONSIDERATIONS

If you are using Docker Compose Wrapper you are trusting your users. This wrapper doesn't provide any security layer: the aim is just to
expose some commands to users in order to permit them to easily deploy and manage well-defined containers or actions.

## Configuration

The wrapper can be easily configured trough some variables defined in the script:

* **dc_confd**: the directory conatining all the docker-compose YAML files
* **command_label_root**: the root label namespace for commands
* **dc_denied_commands**: all the docker-compose commands matching this regex will be denied
* **slack_webook**: the SLACK incoming [webook](https://api.slack.com/incoming-webhooks) for the notification bot, if not configured the SLACK notifications are disabled
* **slack_channel**: the SLACK notification channel
* **slack_botemoji**: the SLACK bot emoji
* **slack_botname**: the SLACK bot name
* **slack_message_prefix**: the SLACK message prefix
* **hipchat_webhook**: The HipChat incoming [webhook](https://www.hipchat.com/docs/apiv2/method/send_room_notification) for the notification bot, if not configured the HipChat notifications are disabled
* **hipchat_message_prefix**: the HipChat message prefix

### Pool definition

In order to define a pool you have to create a docker-compose YAML file into the **dc_confd** directory. The file name will define the pool name (Eg. nginx.yaml will define the nginx pool).
If you want to expose some commands to exec you have to define a label under the **command_label_root** namespace:

The following example defines a pool containing a single container (named nginx1) exposing the **shell** command, executing the **shell** command trough the wrapper will execute **docker exec -it nginx1 /bin/bash**

```yaml
version: '2'
services:
    nginx1:
        image: nginx
        labels:
            management.command.shell: "docker exec -it nginx1 /bin/bash"
        container_name: nginx1
        stdin_open: true
        tty: true
```

### SSH Usage

The common usage scenario is to use this wrapper as an SSH command wrapper adding the *command* parameter to the **authorized_keys**:

```
command="/opt/bin/dcw",no-port-forwarding,no-agent-forwarding,no-X11-forwarding ssh-rsa AAAAB3NzaC1 [..] == pietro@hank
```

## Usage

```
Usage:

./dcw <pool|command> <args>

Examples:

./dcw pool ldap ps

    Run the docker-compose ps over the ldap service pool

./dcw pool ldap start ldap1

    Start the service ldap1 from the ldap pool

./dcw command ldap1 shell

    Execute the command defined into the label 'management.command.shell' of the ldap1 container

./dcw command ldap1 help

    List all the available commands into the container ldap1
```

#### Actions

Action can be *pool* or *command*

##### Pool

The *pool* action requires the pool name, pool action is a simple docker-compose wrapper using the pool-related YAML configuration file, so you can execute all the available docket-compose commands. Trough the **dc_confd** variable you have to configure the directory containing all the docker-compose YAML files.

###### Example:

The following command prints the YAML docker-compose configuration file for the *ldap* pool (executes **docker-compose -f ${dc_confd}/<pool>.yaml**):

```
./dcw pool ldap config
```

The following command starts all the containers of the *ldap* pool:

```
./dcw pool ldap up -d
```

##### Command

The *command* action executes a command defined on a container label. The label name must be into the **action_label_root** namespace:

Container *ldap1* label *management.command.shell*

```
$ docker inspect -f '{{ index .Config.Labels "management.command.shell" }}' ldap1
docker exec -it ldap1 /bin/bash 
```

Executing the *shell* command on the *ldap1* container:

```
./dcw command ldap1 shell
INFO: executing command from label *management.command.shell* into container *ldap1*
root@72b78ab8b5d1:/# 
```
