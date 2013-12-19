# TileIt

a node.js <a href="http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames">slippy</a> maps tile server


Tiles Sources:
  - file: just a bunch of file in folders; format "/path_to_in_config/[mapname]/[z]/[x]/[y].[ext]"
  - memcached: serve files from memory
  - tirex: request & deliver tiles from tirex metatiles renderer (derived from <a href="http://svn.openstreetmap.org/applications/utils/tirex/tileserver/">osm tileserver</a>)
  - tiles: A mirroring map tile proxy (derived from <a href="https://github.com/yetzt/tilethief.git">tilethief</a>)
  - wms: A mirroring map tile proxy for <a href="http://en.wikipedia.org/wiki/Web_Map_Service">wms</a>-server
  - mapnik: render tiles with <a href="https://github.com/mapnik">mapnik</a> (derived from <a href="https://github.com/mapbox/tilelive-mapnik">tilelive-mapnik</a>)

## Setup

execute "npm install" to install the required nodejs-packages


## Configuration

### Plugin Configuration

rename `config.js.dist` to `config.js` and edit the settings with your favorite editor


```json
module.exports = {
	"hostname": "localhost",                        // host adress of the tileserver
	"port": 80,                                     // port of the tileserver
	"log": {
		"path": "./logs", 							// path to logfiles
    	"levels": ["info", "warn", "error", "tiles", "debug"] // set of levels - debug & tiles are logged in extra files
    },
	"configpath": "./maps/enabled_maps",         // load map configs from this path
	"preview": './maps/preview',					// remove this line to disable build-in leaflet preview
	"max_age": 3 * 60 * 60 * 1000,                  // max age header for http-request

	"plugs": {
		"tirex": {
			"enabled": true,				// enable-disable this plug
			"config_dir": "./data/tirex/renderer/",		// path to Tirex config
			"master_socket": "/run/tirex/master.sock",	// path to Tirex Master Unix Datagram Socket
			"timeout": 1000					// how long to wait for Tirex to render/response
		},
		"file": {
			"enabled": true,				// enable-disable this plug
			"path": "./data/xyz.ext/"      // global path for file plug (may be overwritten individually by map config)
		},
		"tiles": {
			"enabled": true,				// enable-disable this plug
			"concurrent_requests": 10       // how many tiles can be process parallel
		},
		"wms": {
			"enabled": true,				// enable-disable this plug
			"concurrent_requests": 10       // how many wms tiles can be process parallel
		},
		"memcached": {
  			"enabled": true,				// enable-disable this plug
			"hosts": "localhost:11211",      // host/hosts & ports of memcached
			"rev": "0",                     // invalidate all tiles by a version number (may be overwritten individually by map config) 
			"prefix": "tiles_",             // prefix map names (may be overwritten individually by map config)
			"expiration": 1000000,          // a tile can be replaced after ms (may be overwritten individually by map config)
			"options": {
				"timeout": 10,                // timeout of a memcached request
				"maxExpiration": 2592000      // maximal timeout of a memcached request
			}
		}
	}
};
```

see <a href="https://github.com/3rd-Eden/node-memcached">node-memcached</a> for more memcached options

### Map Configuration

you can defy one or more maps in a json file, residing in /maps/map-enabled/ (maybe linked there from /maps/map-available) 

or you can build your map configs in javascript

Note: Maps for Tirex are loaded from the Tirex configuration path, so no extra config files are needed. You may overwrite a tirex map by just reusing the map name (eg. for memcaching)

```json
{
	"mapname": {                    //mandatory, mapname is used in tile-url
		"minz": 0,                  //optional, default: "0"
		"maxz": 18,                 //optional, default: "18"
		"format": "png",            //optional, default: "png"
		"source": {
		    "name of plug": { map related options for the source }
		    ...see source definition below...
			  please note, sources are processed from top to bottom, so ordering of sources is important 
		}
	},
	...
}
```

see <a href="https://github.com/ffalt/tileit/tree/master/maps/maps-example">/maps/map-examples/</a> for examples


### Map Source Configuration

#### file

```json
"file": {
	"enabled": true,				// enable-disable this plug
	"path": "./some/demo/path/" //optional, if empty, global path from config.js + mapname is used otherwise
}
```

#### tiles 


```json
"tiles": {
	"enabled": true,				// enable-disable this plug
	"url": "http://tiles.example.org/slippytilemap"  //mandatory
}
```

#### memcached

```json
"mapcached" : {
	"enabled": true,				// enable-disable this plug
	"rev": "0",                     //optional, global rev is used otherwise (see global config above)
	"prefix": "tiles_",             //optional, global prefix is used otherwise (see global config above)
	"expiration": 1000000,          //optional, global expiration is used otherwise (see global config above) 
}
```

#### tirex

```json
"tirex" : {
	"enabled": true				// enable-disable this plug
}
```


## TODO

general: max-age strategy?

memcached: expiration strategy?

tests

