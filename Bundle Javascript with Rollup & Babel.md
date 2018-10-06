# Bundle Javascript with Rollup & Babel

Most browsers don't (fully) support ES6+ Javascript yet. Especially if you are using import/export modules acrross multiple files. [Rollup](https://rollupjs.org/guide/en) is a tool which (among other things) combines muliple Javascript source files into a single .js file (often called a bundle). Less rountrips to the server and more flexible development practices.

Rollup is clever, it uses a process called [Tree Shaking](https://en.wikipedia.org/wiki/Tree_shaking) which removes unused code from your source files. If you are using lots of third party libraries this can potenially reduce filesize dramatically. 

The resultant code will still be ES6+ and will often work in the latest browsers as it. Genreally however you may need to use [Babel](https://babeljs.io) to [transpile](https://en.wikipedia.org/wiki/Source-to-source_compiler) your code to ensure it works across older browers. 

And that is what we are going to do here.

### Getting setup

Start a new project with this folder structure...

```
rollupBableExample
├── build/
|	├── index.html
|	├── js/
├── source/
|   ├── js/
|   	├── index.js
|   	├── myModule.js
```

With these files...

```html
<!-- build/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>proES6</title>
  <link rel="icon" type="image/png" href="data:image/png;base64,iVBORw0KGgo=">
</head>
<body>
<h1>Well hello there</h1>
<script src="js/bundle.js"></script>
</body>
</html>
```

```javascript
//source/js/index.js
import { exportedVariable } from './myModule.js'

console.log(`The exported variable is : ${exportedVariable}`);    
```

```javascript
//source/js/myModule.js
export const exportedVariable = "Whoop";

export const anotherVariable = "goodbye";

export function exportedFunction(number) {
  return `The number is ${number}`;
}
```

### Adding rollup

We need a `package.json` file to manage all the project dependencies and install various tools. Ensure that you have [node](https://nodejs.org/en/) and [yarn](https://yarnpkg.com/lang/en/) installed (you can use [npm](https://www.npmjs.com) but I prefer yarn, it's faster). From within your project folder...

```bash
$ yarn init
```

Answer all the questions, values don't matter too much except for **Entry point** this should be `source/js/index.js`

Now we need to install **Rollup**.

```bash
$ yarn add rollup --dev
```

And it's ready to use. From the command line the format is...

```bash
$ rollup <entry point> --file <output file> --format <type of JS produced>
```

Where `<entry point>` is your main JS file from which all your JS is linked (imported).  `<output file>` is the bundle we want to produce and `--format` is the type of JS we want to produce (be it for Node, Browsers etc).

Our version of rollup was installed in node_modules so it becomes...

```bash
$ node_modules/.bin/rollup source/js/index.js --file build/js/bundle.js --format iife
```

The [iife](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) format is comlicated but for our purposes it means the outputed JS will be browser friendly. 

Take a look at `build/js/bundle.js` It should look something like this...

````javascript
(function () {
  'use strict';

  //source/js/myModule.js
  const exportedVariable = "Whoop";

  //source/js/index.js

  console.log(`The exported variable is : ${exportedVariable}`);

}());
````

You'll notice that our two JS files have been combined into one. Also because of the tree shaking the unused function `exportedFunction(number)` and unsued variable `const anotherVariable` from our module has been exclued. Neat huh?

While the modules have been resolved (no more import/export) and everything is in a single file, the code is still ES6. As mentioned before this may well work in very modern browsers but if you want your code to work acroos a wide range of browsers another step is required.

#### Adding a rollup config file

Just before that though, typing out this line each time is boring and potentially error prone...

```bash
$ node_modules/.bin/rollup source/js/index.js --file build/js/bundle.js --format iife
```

Luckily though we don't need to do it every time. Lets create a `rollup.config.js` file.

````javascript
// rollup.config.js

module.exports = {
  input: 'source/js/index.js',
  output: {
    file: 'build/js/bundle.js',
    format: 'iife'
  }
};
````

With this in place all we have to do is...

```bash
$ node_modules/.bin/rollup -c <config file>
```

If you don't add anything in <config file> it automatically defaults to `rollup.config.js` and runs just as before. However this allows you to have multiple config files such as ``rollup.debug.config.js` or `rollup.production.config.js` if required.

### Adding Babel

Lets install babel and configure it. We are going to need a number of packages...

```bash
$ yarn add @babel/core --dev				// The core babel tools
$ yarn add @babel/preset-env --dev			// Smart presets for babel
$ yarn add rollup-plugin-babel --dev		// The rollup plugin
```

With these installed we need to tell Rollup to use them. We do this inside the `rollup.config.js` file.

```javascript
// rollup.config.js

import babel from 'rollup-plugin-babel'; // import the plugin

module.exports = {
  input: 'source/js/index.js',
  output: {
    file: 'build/js/bundle.js',
    format: 'iife'
  },
  plugins: [
    babel({ exclude: 'node_modules/**' }) // include the plugin
  ]
};
```

We import the plugin and then call it. We've also added a configuration option to ensure that babel doesn't try and transpile the node_modules folder. 

We need to configure Babel before we can use it. Create `.babelrc` in the project root with the following...

````json
// .babelrc
{
  "presets": [
    [
      "@babel/env",
      {
          "modules": false
      }
    ]
  ]
}
````

Now we can run rollup again. 

```bash 
$ node_modules/.bin/rollup -c 
```

Have a look at `build/js/bundle.js` it should be...

```javascript
// build/js/bundle.js
(function () {
  'use strict';

  //source/js/myModule.js
  var exportedVariable = "Whoop";

  //source/js/index.js
  console.log("The exported variable is : ".concat(exportedVariable));

}());
```

As you can see the ES6 template literals have been converted into ES5 (browser safe) javascript. Cool.

### Compressing the bundle

Once we've combined everything into our single script and converted it to browser safe code we may want to minify it. That is make the `build/js/bundle.js` as small as possible. To do this we'll use [Terser](https://github.com/terser-js/terser) via [rolllup-plugin-terser](https://github.com/TrySound/rollup-plugin-terser).

Install it with...

```bash
$ yarn add rollup-plugin-terser --dev
```

And add it to your rollup config file `rollup.config.js`

```javascript
// rollup.config.js
import  babel  from 'rollup-plugin-babel';
import { terser } from 'rollup-plugin-terser'; // Import the plugin

module.exports = {
  input: 'source/js/index.js',
  output: {
    file: 'build/js/bundle.js',
    format: 'iife'
  },
  plugins:[
    babel({ exclude: 'node_modules/**' }),
    terser()                  //Call Terser
  ]
};
```

Run rollup. 

```bash 
$ node_modules/.bin/rollup -c 
```

And `build/js/bundle.js` will look nicely compressed...

```javascript
!function(){"use strict";console.log("The exported variable is : ".concat("Whoop"))}();
```

### One last thing

So far we've just been bundling files from our `source/js` folder. What if we want to use third party modules from `node_modules`? 

To do this lets download a simple `node_module` which we can call from `source/js/index.js`

```bash
$ yarn add the-answer --dev 
```

Then change `source/js/index.js` to...

```javascript
//source/js/index.js
import { exportedVariable } from './myModule.js'
import theAnswer from 'the-answer'

console.log(`The exported variable is : ${exportedVariable} the answer is ${theAnswer}`);  
```

Run rollup. 

```bash 
$ node_modules/.bin/rollup -c 
```

You should get something like...

```bash
> (!) Unresolved dependencies
> https://rollupjs.org/guide/en#warning-treating-module-as-external-dependency
> the-answer (imported by source/js/index.js)
> (!) Missing global variable name
> Use output.globals to specify browser global variable names corresponding to exteral modules
> the-answer (guessing 'theAnswer')
```

That's because Rollup can't find **the-answer** in the node_modules folder. We can fix that with a couple of plugins. 

```bash
$ yarn add rollup-plugin-node-resolve --dev
$ yarn add rollup-plugin-commonjs --dev
```

[rollup-plugin-node-resolve](https://github.com/rollup/rollup-plugin-node-resolve) allows rollup to find modules/libs in the `/node_modules` folder and [rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs) converts [commonJS](https://en.wikipedia.org/wiki/CommonJS) style modules to ES6. 

And then add them to the `rollup.config.js` file. 

```javascript
// rollup.config.js
import  babel  from 'rollup-plugin-babel';
import { terser } from 'rollup-plugin-terser'; 
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';


module.exports = {
  input: 'source/js/index.js',
  output: {
    file: 'build/js/bundle.js',
    format: 'iife'
  },
  plugins:[
    resolve(),
    commonjs(),
    babel({ exclude: 'node_modules/**' }),
    terser()  
  ]
};
```

Run rollup. 

```bash 
$ node_modules/.bin/rollup -c 
```

And have a look at `build/js/bundle.js` you'll end up with...

```javascript
!function(){"use strict";console.log("The exported variable is : ".concat("Whoop"," the answer is ").concat(42))}();
```

And you can see [the answer to life the universe and everything](https://en.wikipedia.org/wiki/42_(number)#The_Hitchhiker's_Guide_to_the_Galaxy) imported from **the-answer** node_module right there in your code. Boom.





