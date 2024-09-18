# Modding for Open-MC

This repo concerns modding for the client.

At the heart of Open-MC, there is no code. This is a core principle of the Component API: All features present in the base game are made using the Component API

The "Component API" describes multiple systems that work together:
- A list of "components" which are loaded together
	- These are used to add new features and functionality
	- This is found in the server's properties.yaml under `components: `
- A list of "resource maps" which replace resource(s) used by components with other ones
	- These are used to modify functionality, textures or sounds of existing components
	- Some components will expose a lot of their internals so that they can be modified without resource maps. Use components if it is available!
	- This is found in the server's properties.yaml under `resourcemaps: `
- All the APIs provided by the built-in component named `core`
	- This automatically loaded and provides a graphics library, physics engine, and a basic network protocol for the game's core functionality.

Another component named `vanilla` is usually specified and includes all the blocks, items, entities, etc present in the base game Minecraft. The server can chose to add additional components or replace this component entirely.

All other components should be specified by URLs, absolute, or relative to the server's address (using the same port and encryption as used to connect, usually encrypted, :27277). For example, a server at `example.org` will be connected to at `wss://example.org:27277/` and, if it loads `/static/mod/index.js`, will be fetched from `https://example.org:27277/static/mod/index.js`.

Files hosted in this manner do not need an appropriate `Content-Type` header, or even one at all, but they do need an appropriate CORS header (`Access-Control-Allow-Origin` set to `*` or to the game client's address). A common hosting place that meets this requirement is github's raw URLs (`https://raw.githubusercontent.com/<username>/<repo>/refs/heads/<branch>/<path_to_file>`, for example, this file is at `https://raw.githubusercontent.com/open-mc/component-api/refs/heads/main/README.md`).

Note that all file imports, component URLs, etc must begin with `/`, `./`, `../`, `http://` or `https://`. Relative paths like `other.js` or `~/icon.png` are not allowed.

Resource maps are specified differently, in the format `<base> <map_url>`. Base specifies the component to map against, and map_url points to a file containing a list of maps, it has this format:
```yaml
# comment
resource_path target_url?
resource_path target_url?
resource_path target_url?
```

Let's say we wanted to make a vanilla resource pack. We want to map against the component `vanilla` (since that's where vanilla resources are). The resource map entry might look like
```
vanilla https://vanilla-plus.net/map.txt
```
And the contents of the map.txt file might look like
```yaml
# This maps vanilla's terrain.png to https://vanilla-plus.net/terrain.png
./terrain.png
# This maps vanilla's items.png to https://vanilla-plus.net/extra/items1.png
./items.png ./extra/items1.png

# This maps vanilla's blocks.js to https://vanilla-plus.net/blocks-patched.js
./blocks.js ./blocks-patched.js

# File shortened for brevity
...
```
Note that the paths on the left are relative to the mapped component `vanilla` and the ones on the right are relative to the `map.txt` file.

> Note: While the resources replaced are relative to `vanilla`, any other component that shares the same resources will receive the changed file too.
> Note 2: If you replace JS files, if the original file imports any other files, those may not be imported anymore. The new file's imports will be imported instead

If you update your component's files frequently, it is strongly recommended that you use versioning. This helps the client to immediately replace cache of old versions so that your players are always using the newest version. For the example above, if map.txt was updated, we would replace `https://vanilla-plus.net/map.txt` with `https://vanilla-plus.net/map.txt^1`, and then `^2`, and then `^3`, etc for every change made. This applies to component URLs as well as the contents of resource map files, javascipt import URLs, cache list files, etc...