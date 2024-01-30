# 创建自定义预设
::: warning
您不应编辑面板随附的现有预设。我们每次对这些预设的更新，在你更新数据库时会覆盖面板的原有预设，也就是说你将丢失这些所有改动的内容。
:::

[[toc]]

您需要做的第一件事是创建一个新的预设组(类似文件夹)。名称和描述就不言而喻了。`预设组名称` _需要确保唯一性_ ，不被其他任何预设组使用，并且只能包含字母、数字、下划线和破折号(中文也是可以的)。这是守护进程存储预设的预设组名称。

预设的默认启动命令也是必需的，但是可以根据变量进行动态更改。

## Create New Option
创建完预设组后, 点击页面右下角的 `新预设` 按钮.

![](../../../.vuepress/public/community/config/eggs/Pterodactyl_Create_New_Egg_Select.png)

大多数配置都将在随后打开的这个预设设置页面中进行. 你要做的第一件事情就是从 `所属预设组` 下拉框中选择你的预设属于哪个预设租.

![](../../../.vuepress/public/community/config/eggs/Pterodactyl_Create_New_Egg_Process_Management.png)

随后为你的预设起一个名字并填写在 `预设名` 中, 在这个样例中我使用了 `Widget` 这个名字. 你还需要提供一个有效的docker镜像和启动命令 (启动命令在具体的服务器创建后可以单独针对此服进行再次编辑).

_Docker images 必须是专门设计用于翼龙面板的那些._ 你可以在我们的
 [创建docker镜像](/community/config/eggs/creating_a_custom_image.md) 指南中阅读更多相关信息.

## Configure Process Management
This is perhaps the most important step in this service option configuration, as this tells the Daemon how to run everything.

![](../../../.vuepress/public/community/config/eggs/Pterodactyl_Create_New_Egg_Process_Management.png)

The first field you'll encounter is `Copy Settings From`. The default selection is `None`. That is expected, and okay.
This dropdown is discussed at the end of this article.

### Stop Command
Next, you'll encounter `Stop Command` and, as the name implies, this should be the command used to safely stop the
option. For some games, this is `stop` or `end`. Certain programs and games don't have a specified stop command, so
you can enter `^C` to have the daemon execute a `SIGINT` to end the process.

### Log Storage
Logs are competely handeled by the daemon now and use the docker logs to output the complete output from the server.
This can be set like below.

```json
{}
``` 

### Configuration Files
The next block is one of the most complex blocks, the `Configuration Files` descriptor. The Daemon will process this
block prior to booting the server to ensure all of the required settings are defined and set correctly.

```json
{
    "server.properties": {
        "parser": "properties",
        "find": {
            "server-ip": "0.0.0.0",
            "enable-query": "true",
            "server-port": "{{server.build.default.port}}",
            "query.port": "{{server.build.default.port}}"
        }
    }
}
```

In this example, we are telling the Daemon to read `server.properties` in `/home/container`. Within this block, we
define a `parser`, in this case `properties` but the following are [valid parsers](https://github.com/pterodactyl-china/wings/blob/develop/parser/parser.go#L25-L30):

* `file` — This parser goes based on matching the beginning of lines, and not a specific property like the other five.
Avoid using this parser if possible.
* `yaml` (supports `*` notation)
* `properties`
* `ini`
* `json` (supports `*` notation)
* `xml`

Once you have defined a parser, we then define a `find` block which tells the Daemon what specific elements to find
and replace. In this example, we have provided four separate items within the `server.properties` file that we want to
find and replace to the assigned values. You can use either an exact value, or define a specific server setting from
the `server.json` file. In this case, we're assigning the default server port to be used as the `server-port` and
`query.port`. **These placeholders are case sensitive, and should have no spaces in them.**

You can have multiple files listed here, the Daemon will process them in parallel before starting the server. When
using `yaml` or `json` you can use more advanced searching for elements.

```json
{
    "config.yml": {
        "parser": "yaml",
        "find": {
            "listeners[0].query_enabled": true,
            "listeners[0].query_port": "{{server.build.default.port}}",
            "listeners[0].host": "0.0.0.0:{{server.build.default.port}}",
            "servers.*.address": {
                "127.0.0.1": "{{config.docker.interface}}",
                "localhost": "{{config.docker.interface}}"
            }
        }
    }
}
```

In this example, we are parsing `config.yml` using the `yaml` parser. The first three find items are simply assigning
ports and IPs for the first listener block. The last one, `servers.*.address` uses wildcard matching to match any items
within the `servers` block, and then finding each `address` block for those items.

::: v-pre
An advanced feature of this file configuration is the ability to define multiple find and replace statements for a
single matching line. In this case, we are looking for either `127.0.0.1` or `localhost` and replacing them with the
docker interface defined in the configuration file using `{{config.docker.interface}}`. 
:::

### Start Configuration
The last block to configure is the `Start Configuration` for servers running using this service option.

```json
{
    "done": ")! For help, type "
}
```

In the example block above, we define `done` as the entire line, or part of a line that indicates a server is done
starting, and is ready for players to join. When the Daemon sees this output, it will mark the server as `ON` rather
than `STARTING`.

That concludes basic service option configuration.

## Copy Settings From
As mentioned above, there is a unique `Copy Settings From` dropdown when adding a new option. This gives you the
ability to, as the name suggests, copy settings defined above from a different option.

![](../../../.vuepress/public/community/config/eggs/Pterodactyl_Create_New_Egg_Copy_Settings_From.png)

In the panel, we use this to copy settings that remain the same between similar service options, such as many of the
Minecraft options.

For example, lets look at the `Sponge (SpongeVanilla)` service option.

As you can see, it as been told to copy settings from `Vanilla Minecraft`. This means that any of the fields that are
left blank will inherit from the assigned parent. We then define a specific `userInteraction` line that is different in
Sponge compared to Vanilla, but tell it that everything else should remain the same.

*Please note that `Copy Settings From` does not support nested copies, you can only copy from a single parent,
and that parent **must not be copying from another option.***

## Egg Variables
One of the great parts of the Egg Variables is the ability to define specific variables that users and/or admins can
control to tweak different settings without letting users modify the startup command. To create new variables, or edit
existing ones, visit the new service option you created, and click the `Variables` tab at the top of the page. Lets take
a look at an example variable that we can create.

![](../../../.vuepress/public/community/config/eggs/Pterodactyl_Create_New_Egg_Variables.png)

::: v-pre
The name and description are rather self-explanitory, so I'll skip down to the `Environment Variable` box. This should
be an Alpha-Numeric name with underscores, and should be uppercase. This will be the name of the environment variable
which can be accessed in the startup command as `{{WOOZLE_WOO}}`, within file modifications as `{{env.WOOZLE_WOO}}`, or
just `${WOOZLE_WOO}` in any shell scripts (it is passed through in the environment). We also define a default value for
this environment variable in this example, but it is not required to do so.
:::

The next section is `Permissions`, which is a dropdown with two options: `Users Can View` and `Users Can Edit`.

* `Users Can View` — allows a user to view the field on the front-end, as well as the assigned value of that variable.
They will be able to see it replaced in their startup command.
* `Users Can Edit` — allows a user to edit the value of the variable, for example the name of their `server.jar` file
if running Minecraft.

You should use caution here, even if you assign neither of the permissions it does not mean that the value will be
hidden. Crafty users will still be able to get the environment on their server. In most cases this is simply hiding
it from the user, and then used within the Dockerfile to perform actions, thus it is not important for the user to see.

Finally, you will need to define some input rules to validate the value against. In this example, we use
`required|string|between:1,10`, which means the field is `required`, must be a `string`, and must be between `1` and
`10` characters in length. You can find [all of the available validation rules](https://laravel.com/docs/5.6/validation#available-validation-rules)
on the Laravel website. You can also use ReGEX based validation by using the `regex:` rule flag. For example,
[`required|regex:/^([\w\d._-]+)(\.jar)$/`](https://regex101.com/r/k4oEOn/1) will require the field, and will match the
regex as any letters or numbers (`\w\d`) including underscore (`_`), periods (`.`), and dashes (`-`) ending in `.jar`.

They will then be visible when managing the startup for a server in both the Admin CP and on the Front-End.

![](../../../.vuepress/public/community/config/eggs/Pterodactyl_Create_New_Egg_Startup.png)

## List of default variables

The default variables are always accessible to all eggs and don't have to be created separately. They can be used in the egg startup, install script, or the configuration file parser.

| Variable | Description | Example |
|----------|-------------|---------|
| TZ       | Time Zone |  `Etc/UTC` |
| STARTUP  | Startup command of the egg | `java -Xms128M -Xmx{{SERVER_MEMORY}}M -jar {{SERVER_JARFILE}}` |
| SERVER_MEMORY | Memory available for the server in MB | `512` |
| SERVER_IP | Default ip of the server | `127.0.0.1` |
| SERVER_PORT | Primary Server Port | `27015` |
| P_SERVER_LOCATION | Location of the server | `Example City` |
| P_SERVER_UUID | UUID of the server | `539fdca8-4a08-4551-a8d2-8ee5475b50d9` |
| P_SERVER_ALLOCATION_LIMIT | Limit of allocations allowed for the server | `0` |
