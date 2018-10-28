### Dev

```
watchify -vd -p browserify-hmr -e src/main.js -o index.js
```

- `watchify`: runs browserify and watches for changes  
- `-v`: verbose logging. Tells you stuff if stuff breaks  
- `-d`: "debug" includes a sourcemap into the bundle  
- `-p browserify-hmr`: includes "hot module reloading" logic in bundle  
- `-e src/main.js`: entrypoint  
- `-o index.js`: output file  


### Build

```
cross-env NODE_ENV=production browserify -g envify -p [ vueify/plugins/extract-css -o index.css ] -p bundle-collapser/plugin -e src/main.js | uglifyjs -c warnings=false -m > index.js
```

- `cross-env NODE_ENV=production`: sets the environment variable "NODE_ENV" to production no matter what OS you are using  
- `browserify`: bundles the script  
- `-g envify`: not necessarily needed, but [good practice](https://github.com/hughsk/envify)  
- `-p [ vueify/plugins/extract-css -o index.css ]`: extracts CSS from the JS bundle and writes it to the `index.css` file  
- `-p bundle-collapser/plugin`: optional, saves a few bytes by replacing file paths in the bundle with numbers  
- `-e src/main.js`: entrypoint  
- `| uglifyjs -c warnings=false -m`: pipes the bundled code into uglifyjs to minimize it (`-m` "mangles" variable names, compressing the bundle more)  
- `> index.js`: redirects the output to the `index.js` file


### Globally applied plugins

```
  "browserify": {
    "transform": [
      "babelify",
      "vueify"
    ]
  },
```

babelify and vueify are applied to both commands:

- Babelify transforms modern JS to old JS. 
- Vueify enables the import of `.vue` files.

babelify also requires a config file (`.babelrc`):

```
{
  "presets": [
    "env"
  ]
}
```

In theory, instead of the `.babelrc` file, this should also work:

```
"browserify": {
    "transform": [
      [ "babelify", { "presets": ["env"] } ],
      "vueify"
    ]
  },
```