Community/slack-gathered tips for working on kirby plugins. <br/>
Unofficial documentation for Kirby CMS. It is **not** maintained by the Kirby team.

- [Plugin architecture](#plugin-architecture)
  * [Build process](#build-process)
- [Fields](#fields)
  * [Work with your field's value](#work-with-your-fields-value)
  * [Tell Kirby there's new content to save](#tell-kirby-theres-new-content-to-save)
- [Multi-language](#multi-language)
  * [i18n](#i18n)
  * [Listen to language change](#listen-to-language-change)

<br/>


## Plugin architecture

#### Build process

- Add this `package.json` (annotated version [here](extras/annotated-package.md)) in the root of your plugin folder:

```json
{
  "scripts": {
    "dev": "watchify -vd -p browserify-hmr -e src/main.js -o index.js",
    "build": "cross-env NODE_ENV=production browserify -g envify -p [ vueify/plugins/extract-css -o index.css ] -p bundle-collapser/plugin -e src/main.js | uglifyjs -c warnings=false -m > index.js"
  },
  "browserify": {
    "transform": [
      "babelify",
      "vueify"
    ]
  },
  "devDependencies": {
    "babel-core": "^6.26.3",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-preset-env": "^1.7.0",
    "babelify": "^8.0.0",
    "browserify": "^16.2.2",
    "browserify-hmr": "^0.3.6",
    "bundle-collapser": "^1.3.0",
    "cross-env": "^5.2.0",
    "envify": "^4.1.0",
    "uglify-js": "^3.4.7",
    "vue": "^2.5.17",
    "vueify": "^9.4.1",
    "watchify": "^3.11.0"
  }
}
```

- Add a `.babelrc` in the root of your plugin folder:

```json
{
  "presets": [
    "env"
  ]
}
```

- Adjust your plugin's architecture:
  * Move what's within `index.js` to `src/main.js`. It will be automatically extracted.
  * Move what's within `index.css` to `<style>` tags in your JS components. It will be automatically extracted.

- Install `npm` if needed. Run `npm install` to get all dependencies.

- Use the following commands:
  * `npm run dev` when working on the plugin, live updates and hot-reload.
  * `npm run build` mandatory to publish your plugin. Will extract js and css.


<br/>

## Fields

#### Work with your field's value

To work with your field’s saved value you need a `value` prop in your `index.js`:

```javascript
props: {
    value: {
        type: [String, Boolean, Number, Object, Array]
    }
}
```

#### Tell Kirby there's new content to save

Having an 'input' method, that you call with `@input` will do most of the job:

```javascript
methods: {
    input(data) {
        this.$emit("input", data);
    }
}
```

In the template:

```html
<k-text-field v-model="value" name="seo" label="Boring text" @input="input"/>
```

<br/>

## Multi-language

#### i18n

Worth mentioning: the localization plugin isn't `vue-i18n` but [vuex-i18n](https://github.com/dkfbasel/vuex-i18n). 
The syntax isn't the same, for example to pluralize strings:

```javascript
$t('pluralized.string', number) // vue-i18n, number is the 2nd argument
$t('pluralized.string', {}, number) // vuex-i18n, number is the 3rd argument
```

#### Listen to language change

Here's a snippet to get the current language code (remove `.code` to get the whole language object) and listen to page language change on multi-language websites:

```javascript
computed: {
    currentLanguage() {
        return this.$store.state.languages.current.code // returns en, fr, ...
    }
},
watch: {
    currentLanguage() {
        // do your thing
    }
},
```