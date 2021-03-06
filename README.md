# mirum-vue-form-generator [![NPM version](https://img.shields.io/npm/v/vue-form-generator.svg)](https://www.npmjs.com/package/vue-form-generator) ![VueJS v2.x compatible](https://img.shields.io/badge/vue%202.x-compatible-green.svg)

Forked from [vue-form-generator](https://github.com/vue-generators/vue-form-generator/tree/v2.3.3) v2.3.3

A schema-based form generator component for Vue.js.

## Demo

[JSFiddle simple example](https://jsfiddle.net/zoul0813/d8excp36/)

[CodePen simple example](https://codepen.io/zoul0813/pen/OrNVNw)

[![Screenshot](https://github.com/vue-generators/vue-form-generator-docs/raw/master/assets/vfg-example1.png)](https://jsfiddle.net/zoul0813/d8excp36/)

## Features

*   reactive forms based on schemas
*   multiple object editing
*   21 field types
*   built-in validators
*   core & full bundles (11kb and 19kb gzipped)
*   Bootstrap friendly templates
*   customizable styles
*   can be extended easily with custom fields
*   ...etc

## Documentation

[Online documentation on Gitbook](https://vue-generators.gitbook.io/vue-generators/)

## Dependencies

vue-form-generator uses [fecha](https://github.com/taylorhakes/fecha) and [lodash](https://lodash.com/) internally.

While built-in fields don't need external dependencies, optional fields may need other libraries.
These dependencies fall into two camps: jQuery or Vanilla. You can find almost the same functionality in both flavors.
In the end, it's your choice to depend on jQuery or not.

You can find details about dependencies in the official [documentation](https://vue-generators.gitbook.io/vue-generators/) under each specific component.

## Installation

### NPM

You can install it via [NPM](http://npmjs.org/) or [yarn](https://yarnpkg.com/).

### Manual

Download zip package and unpack and add the vfg.css and vfg.js file to your project from dist folder.

```
https://github.com/vue-generators/vue-form-generator/archive/master.zip
```

### Core vs Full version

VueFormGenerator come in two version : `core` and `full`.
Core is a more minimal version with only half the fields.
Full is core + other fields.

*   Full bundle: 75 kB (gzipped: 19 kB)
*   Core bundle: 39 kB (gzipped: 11 kB)

If you don't know what to choose, don't worry, the full is the default version.
If you want the slim down version, here is the changes:

```js
// the "full" way
<script>
  import VueFormGenerator from "vue-form-generator";
  import "vue-form-generator/dist/vfg.css";  // optional full css additions
</script>

// the "core" way
<script>
  import VueFormGenerator from "vue-form-generator/dist/vfg-core.js";
  import "vue-form-generator/dist/vfg-core.css";  // optional core css additions
</script>
```

## Usage

```html
<template>
  <div class="panel-body">
    <vue-form-generator :schema="schema" :model="model" :options="formOptions"></vue-form-generator>
  </div>
</template>

<script>
import VueFormGenerator from "vue-form-generator";

Vue.use(VueFormGenerator);

export default {
  data: {

    model: {
      id: 1,
      name: "John Doe",
      password: "J0hnD03!x4",
      skills: ["Javascript", "VueJS"],
      email: "john.doe@gmail.com",
      status: true
    },

    schema: {
      fields: [{
        type: "input",
        inputType: "text",
        label: "ID (disabled text field)",
        model: "id",
        readonly: true,
        disabled: true
      },{
        type: "input",
        inputType: "text",
        label: "Name",
        model: "name",
        placeholder: "Your name",
        featured: true,
        required: true
      },{
        type: "input",
        inputType: "password",
        label: "Password",
        model: "password",
        min: 6,
        required: true,
        hint: "Minimum 6 characters",
        validator: VueFormGenerator.validators.string
      },{
        type: "select",
        label: "Skills",
        model: "skills",
        values: ["Javascript", "VueJS", "CSS3", "HTML5"]
      },{
        type: "input",
        inputType: "email",
        label: "E-mail",
        model: "email",
        placeholder: "User's e-mail address"
      },{
        type: "checkbox",
        label: "Status",
        model: "status",
        default: true
      }]
    },

    formOptions: {
      validateAfterLoad: true,
      validateAfterChanged: true
    }
  }
}
</script>
```

Usage in local components

```js
import VueFormGenerator from "vue-form-generator";

//component javascript
export default {
	components: {
		"vue-form-generator": VueFormGenerator.component
	}
};
```

## Development

This command will start a `webpack-dev-server` with content of `dev` folder.

```bash
npm run dev
```

## Build

This command will build a distributable version in the `dist` directory.

```bash
npm run build
```

## Test

```bash
npm test
```

or

```bash
npm run ci
```

## Tagging
1. Change version in `package.json` and `package-lock.json`
2. BUILD!
3. Push changes to origin
4. Go to `Releases`
5. Click `Draft a new release`
6. Increase tag version (`v1.0.9`)
7. Click `Publish release`

### Public Custom Fields

* [vue-tel-input](https://github.com/EducationLink/vue-tel-input) - International Telephone Input Boilerplate with Vue (integrated with VueFormGenerator).
* [vfg-field-sourcecode](https://github.com/gwenaelp/vfg-field-sourcecode) - A source code field for vue-form-generator
* [vfg-field-array](https://github.com/gwenaelp/vfg-field-array) - A vue-form-generator field to handle arrays of items of any type.
* [vfg-field-object](https://github.com/gwenaelp/vfg-field-object) - A vue-form-generator field to handle objects, with or without schemas.

## License

vue-form-generator is available under the [MIT license](https://tldrlegal.com/license/mit-license).

## Contact

Copyright (C) 2017 Icebob

[![@icebob](https://img.shields.io/badge/github-icebob-green.svg)](https://github.com/icebob) [![@icebob](https://img.shields.io/badge/twitter-Icebobcsi-blue.svg)](https://twitter.com/Icebobcsi)
