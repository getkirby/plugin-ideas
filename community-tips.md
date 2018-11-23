Community/slack-gathered tips for working on kirby plugins. <br/>
Unofficial documentation for Kirby CMS. It is **not** maintained by the Kirby team.

Last edit: 23.11.2018. K3 moves fast, some things may be outdated.

- [Build processes](#build-processes)
  * [Browserify](#browserify)
  * [Parcel](#parcel)
- [Fields & Sections](#fields--sections)
  * [Get / sync content with other field(s)](#get--sync-content-with-other-fields)
- [Fields](#fields)
  * [Work with your field's value](#work-with-your-fields-value)
  * [Tell Kirby there's new content to save](#tell-kirby-theres-new-content-to-save)
- [Multi-language](#multi-language)
  * [i18n](#i18n)
  * [Listen to language change](#listen-to-language-change)

<br/>


## Build processes

See [this Slack discussion](https://getkirby.slack.com/archives/CBY2BA82E/p1542815307052700) for more details about build processes. **Browserify** seems to be the most reliable at the moment.

#### Browserify

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


#### Parcel

Check out [this boilerplate repo](https://github.com/medienbaecker/kirby3-boilerplate) (may have some pitfalls, check the above Slack thread).

<br/>

## Fields & Sections

#### Get / sync content with other field(s)

Let's say you have a section `mysection` (works with a field too) and a field `myfield` in the same page. You want `mysection` to listen to `myfield`, sync its value and update it if needed.

- Mention in your blueprint the `key` of the field you'd like to listen to:

```yaml
mysection:
  label: Custom section
  type: mysection
  sync: myfield

myfield:
  label: Message
  type: text
```

- In your section's `index.php`, pass the `sync` key as a props:

```php
Kirby::plugin('username/mysection', array(
    'sections' => array(
        'mysection' => array(
            'props' => array(
                'sync' => function($sync = null) {
                    return $sync;
                },
                // ...
            ),
        ),
    ),
);
```

- And fetch it in your `index.js`:

```javascript
export default {
    props: {
        sync:   String,
        parent: String,
        name:   String,
    },
    created() {
        this.$api
            .get(this.parent + "/sections/" + this.name)
            .then(response => {
                this.sync = response.sync
            })
    }
}
```
> **Note:** The `parent | name` and API call is needed to set props for sections until the `$api.load()` helper [is fixed](https://github.com/k-next/kirby/issues/1062).

- You now need to fetch the stored values of the whole page. To do so, in `index.js`, you'll first need to get the `id` of the page's form, then get its values ([ref](https://github.com/k-next/kirby/blob/6221a6b5b8d42aa4c04e6b01c642b8d52be64e36/panel/src/components/Sections/FieldsSection.vue#L30-L38)):

```javascript
export default {
    computed: {
        id() {
            return this.$store.state.form.current
        },
        pageValues() {
            return this.$store.getters["form/values"](this.id)
        },
    },
}
```

- Add another `computed()` data to get only the value of the required field:

```javascript
export default {
    computed: {
        // ...
        syncedValue() {
            return this.sync && this.pageValues ? this.pageValues[this.sync] : false 
        }
    },
}
```

- Optional: if you want to react on this value's changes, you'll need to watch it (`immediate: true` means we want to trigger the `handler()` function on first load):

```javascript
export default {
    watch: {
        syncedValue: {
            immediate: true,
            handler() {
                if(!this.syncedValue) return false
                this.updatedSyncedValue()
            }
        }
    },
    methods: {
        updatedSyncedValue() {
            // do something with this.syncedValue
        }
    }
}
```

- And if you want to change the field's value, you'll need to dispatch an `update` action (arguments are: `[id of the page's form, field name (string), new value]`, [ref](https://github.com/k-next/kirby/blob/6221a6b5b8d42aa4c04e6b01c642b8d52be64e36/panel/src/components/Sections/FieldsSection.vue#L54-L56)):

```javascript
export default {
    methods: {
        setSyncedFieldValue() {
            if(this.sync) {
                this.$store.dispatch("form/update", [this.id, this.sync, 'This is the new message'])
            }
        }
    }
}
```

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

If you are storing an `Array`, you need to explicitely state in your php props that you want to work with a decoded value:

```php
'props' => array(
    'value' => function ($value = null) {
        return Yaml::decode($value);
    }
),
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
        let current = this.$store.state.languages.current
        return current ? current.code : false // returns en, fr, ... if multi-language
    }
},
watch: {
    currentLanguage() {
        // do your thing
    }
},
```
