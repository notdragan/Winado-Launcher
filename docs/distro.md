# Distribution Index

You can use [Nebula](https://github.com/dscalzi/Nebula) to automate the generation of a distribution index.

The distribution index is written in JSON. The general format of the index is as posted below.

```json
{
    "version": "1.0.0",
    "discord": {
        "clientId": "12334567890123456789",
        "smallImageText": "Winado",
        "smallImageKey": ""
    },
    "rss": "",
    "servers": [
        {
            "id": "Winado",
            "name": "Winado PvP/Factions",
            "description": "Winado PvP/Factions, mais attention, la tempête approche :)",
            "icon": "https://cdn.discordapp.com/attachments/729600420225220628/735586634308780062/SealCircle.png",
            "version": "1.0.0",
            "address": "play.winado.fun",
            "minecraftVersion": "1.12.2",
            "discord": {
                "shortId": "Winado",
                "largeImageText": "Winado PvP/Factions",
                "largeImageKey": "server-prod"
            },
            "mainServer": true,
            "autoconnect": true,
            "modules": [
                "Module Objects Here"
            ]
        }
    ]
}
```

## Distro Index Object

#### Example
```JSON
{
    "version": "1.0.0",
    "discord": {
        "clientId": "12334567890123456789",
        "smallImageText": "Winado",
        "smallImageKey": ""
    },
    "rss": "",
    "servers": []
}
```

### `DistroIndex.version: string/semver`

The version of the index format. Will be used in the future to gracefully push updates.

### `DistroIndex.discord: object`

Global settings for [Discord Rich Presence](https://discordapp.com/developers/docs/rich-presence/how-to).

**Properties**

* `discord.clientId: string` - Client ID for th Application registered with Discord.
* `discord.smallImageText: string` - Tootltip for the `smallImageKey`.
* `discord.smallImageKey: string` - Name of the uploaded image for the small profile artwork.


### `DistroIndex.rss: string/url`

A URL to a RSS feed. Used for loading news.

---

### `Server.id: string`

The ID of the server. The launcher saves mod configurations and selected servers by ID. If the ID changes, all data related to the old ID **will be wiped**.

### `Server.name: string`

The name of the server. This is what users see on the UI.

### `Server.description: string`

A brief description of the server. Displayed on the UI to provide users more information.

### `Server.icon: string/url`

A URL to the server's icon. Will be displayed on the UI.

### `Server.version: string/semver`

The version of the server configuration.

### `Server.address: string/url`

The server's IP address.

### `Server.minecraftVersion: string`

The version of minecraft that the server is running.

### `Server.discord: object`

Server specific settings used for [Discord Rich Presence](https://discordapp.com/developers/docs/rich-presence/how-to).

**Properties**

* `discord.shortId: string` - Short ID for the server. Displayed on the second status line as `Server: shortId`
* `discord.largeImageText: string` - Ttooltip for the `largeImageKey`.
* `discord.largeImageKey: string` - Name of the uploaded image for the large profile artwork.

### `Server.mainServer: boolean`

Only one server in the array should have the `mainServer` property enabled. This will tell the launcher that this is the default server to select if either the previously selected server is invalid, or there is no previously selected server. If this field is not defined by any server (avoid this), the first server will be selected as the default. If multiple servers have `mainServer` enabled, the first one the launcher finds will be the effective value. Servers which are not the default may omit this property rather than explicitly setting it to false.

### `Server.autoconnect: boolean`

Whether or not the server can be autoconnected to. If false, the server will not be autoconnected to even when the user has the autoconnect setting enabled.

### `Server.modules: Module[]`

An array of module objects.

---

## Module Object

A module is a generic representation of a file required to run the minecraft client.

#### Example

The parent module will be stored maven style, it's destination path will be resolved by its id. The sub module has a declared `path`, so that value will be used.

### `Module.id: string`

The ID of the module. All modules that are not of type `File` **MUST** use a maven identifier. Version information and other metadata is pulled from the identifier. Modules which are stored maven style use the identifier to resolve the destination path. If the `extension` is not provided, it defaults to `jar`.

**Template**

`my.group:arifact:version@extension`

`my/group/artifact/version/artifact-version.extension`

**Example**

`net.minecraft:launchwrapper:1.12` OR `net.minecraft:launchwrapper:1.12@jar`

`net/minecraft/launchwrapper/1.12/launchwrapper-1.12.jar`

If the module's artifact does not declare the `path` property, its path will be resolved from the ID.

### `Module.name: string`

The name of the module. Used on the UI.

### `Module.type: string`

The type of the module.

### `Module.required: Required`

**OPTIONAL**

Defines whether or not the module is required. If omitted, then the module will be required. 

Only applicable for modules of type:
* `ForgeMod`
* `LiteMod`
* `LiteLoader`


### `Module.artifact: Artifact`

The download artifact for the module.

### `Module.subModules: Module[]`

**OPTIONAL**

An array of sub modules declared by this module. Typically, files which require other files are declared as submodules. A quick example would be a mod, and the configuration file for that mod. Submodules can also declare submodules of their own. The file is parsed recursively, so there is no limit.


## Artifact Object

The format of the module's artifact depends on several things. The most important factor is where the file will be stored. If you are providing a simple file to be placed in the root directory of the client files, you may decided to format the module as the `examplefile` module declared above. This module provides a `path` option, allowing you to directly set where the file will be saved to. Only the `path` will affect the final downloaded file.

Other times, you may want to store the files maven-style, such as with libraries and mods. In this case you must declare the module as the example artifact above. The module `id` will be used to resolve the final path, effectively replacing the `path` property. It must be provided in maven format. More information on this is provided in the documentation for the `id` property.

The resolved/provided paths are appended to a base path depending on the module's declared type.

| Type | Path |
| ---- | ---- |
| `ForgeHosted` | ({`commonDirectory`}/libraries/{`path` OR resolved}) |
| `LiteLoader` | ({`commonDirectory`}/libraries/{`path` OR resolved}) |
| `Library` | ({`commonDirectory`}/libraries/{`path` OR resolved}) |
| `ForgeMod` | ({`commonDirectory`}/modstore/{`path` OR resolved}) |
| `LiteMod` | ({`commonDirectory`}/modstore/{`path` OR resolved}) |
| `File` | ({`instanceDirectory`}/{`Server.id`}/{`path` OR resolved}) |

The `commonDirectory` and `instanceDirectory` values are stored in the launcher's config.json.

### `Artifact.size: number`

The size of the artifact.

### `Artifact.MD5: string`

The MD5 hash of the artifact. This will be used to validate local artifacts.

### `Artifact.path: string`

**OPTIONAL**

A relative path to where the file will be saved. This is appended to the base path for the module's declared type.

If this is not specified, the path will be resolved based on the module's ID.

### `Artifact.url: string/url`

The artifact's download url.

## Required Object

### `Required.value: boolean`

**OPTIONAL**

If the module is required. Defaults to true if this property is omited. 

### `Required.def: boolean`

**OPTIONAL**

If the module is enabled by default. Has no effect unless `Required.value` is false. Defaults to true if this property is omited. 

---

## Module Types

### ForgeHosted

The module type `ForgeHosted` represents forge itself. Currently, the launcher only supports forge servers, as vanilla servers can be connected to via the mojang launcher. The `Hosted` part is key, this means that the forge module must declare its required libraries as submodules.

Ex.

```json
{
    "id": "net.minecraftforge:forge:1.12.2-14.23.5.2854",
    "name": "Minecraft Forge 1.12.2-14.23.5.2854",
    "type": "ForgeHosted",
    "artifact": {
        "size": 4450992,
        "MD5": "3fcc9b0104f0261397d3cc897e55a1c5",
        "url": "http://files.minecraftforge.net/maven/net/minecraftforge/forge/1.12.2-14.23.5.2854/forge-1.12.2-14.23.5.2854-universal.jar"
    },
    "subModules": [
        {
            "id": "net.minecraft:launchwrapper:1.12",
            "name": "Mojang (LaunchWrapper)",
            "type": "Library",
            "artifact": {
                "size": 32999,
                "MD5": "934b2d91c7c5be4a49577c9e6b40e8da",
                "url": "http://mc.westeroscraft.com/WesterosCraftLauncher/files/1.11.2/launchwrapper-1.12.jar"
            }
        }
    ]
}
```

All of forge's required libraries are declared in the `version.json` file found in the root of the forge jar file. These libraries MUST be hosted and declared a submodules or forge will not work.

There were plans to add a `Forge` type, in which the required libraries would be resolved by the launcher and downloaded from forge's servers. The forge servers are down at times, however, so this plan was stopped half-implemented.

---

### LiteLoader

The module type `LiteLoader` represents liteloader. It is handled as a library and added to the classpath at runtime. Special launch conditions are executed when liteloader is present and enabled. This module can be optional and toggled similarly to `ForgeMod` and `Litemod` modules.

---

### Library

The module type `Library` represents a library file which will be required to start the minecraft process. Each library module will be dynamically added to the `-cp` (classpath) argument while building the game process.

---

### ForgeMod

The module type `ForgeMod` represents a mod loaded by the Forge Mod Loader (FML). These files are stored maven-style and passed to FML using forge's [Modlist format](https://github.com/MinecraftForge/FML/wiki/New-JSON-Modlist-format).

---

### LiteMod

The module type `LiteMod` represents a mod loaded by liteloader. These files are stored maven-style and passed to liteloader using forge's [Modlist format](https://github.com/MinecraftForge/FML/wiki/New-JSON-Modlist-format). Documentation for liteloader's implementation of this can be found on [this issue](http://develop.liteloader.com/liteloader/LiteLoader/issues/34).

---

### File

The module type `file` represents a generic file required by the client, another module, etc. These files are stored in the server's instance directory.
