# Notes on Modern JavaScript Project Workflow

These notes are derived from the Code Realm YouTube Channel's React / JS Project Lifecycle playlist; I have no affiliation with the channel. These are just personal notes, and I am amenable to takedown requests. In any case, I highly recommend you pop over to YouTube and check out his channel.

These notes are more focused on the vanilla JavaScript aspects of the workflow, but for the sake of completeness I suppose some of the React-related notes will be included. I'm not focused on recreating the React CSS Spinners project he did, but instead I'm just documenting the overall process.

[Link to Code Realm's YouTube Channel.](https://www.youtube.com/channel/UCUDLFXXKG6zSA1d746rbzLQ)

[Link to the YouTube playlist.](https://www.youtube.com/watch?v=nLwqM034Jjs&list=PLcCp4mjO-z99IPNCrhEyrZimdUG5QXjPd)

---

## Brief Notes on Git, Github, & NPM

This isn't related to the topic at hand, so much as documentation of a brief problem/mistake that I sometimes admittedly create by accident when creating new GitHub Repositories.

After creating a new repository on GitHub with a README & a LICENSE, I tend to accidentally copy the HTTPS location for cloning the repository - instead of the SSH location. The issue then is that inevitably I'm asked at the command line for a username and password when I'm trying to push updates to GitHub, which is not what I want.

It's not hard or impossible to fix this, which is easily done via the following GitHub guide:

[Switching Remote URLs from HTTPS to SSH](https://help.github.com/en/github/using-git/changing-a-remotes-url#switching-remote-urls-from-https-to-ssh)

In summary:

1. List existing remotes

```git
$ git remote -v
> origin  https://github.com/USERNAME/REPOSITORY.git (fetch)
> origin  https://github.com/USERNAME/REPOSITORY.git (push)
```

2. Change remote's URL from HTTPS to SSH

```git
$ git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```

3. Check to see if you did it correctly

```git
$ git remote -v
> origin  git@github.com:USERNAME/REPOSITORY.git (fetch)
> origin  git@github.com:USERNAME/REPOSITORY.git (push)
```

After cloning the project to development environment, in project root initialize npm.

```sh
npm init -y
```

Inside `package.json`, change the license field if you like, and add some keyword strings to the keywords array. After customizing the package.json, push changes to GitHub.

---

## ESLint, Prettier, EditorConfig

In the project root, create a `.gitignore` file and make sure to include `node_modules`.

In the same folder, create a `.editorconfig` file. Below is Code Realm's config that I mostly adopted, but you can also look at [the EditorConfig Website](https://editorconfig.org/) as well.

```js
root = true

[*]

charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
trim_trailing_white_space = true
```

Code Realm seems to prefer the [StandardJS Code Style Format](https://standardjs.com/), which is also what I've elected to go with currently. There are other notable choices out there that I'm ignoring, such as the AirBNB style guide. In any case, Code Realm doesn't use the actual StandardJS Linter, but instead implements it via a combination of [ESLint](https://eslint.org/) and [Prettier](https://prettier.io/) in VS Code.

1. Setup ESLint as a development dependency in the project.

```sh
npm install eslint --save-dev
```

**or**

```sh
npm i -D eslint
```

2. Initialize the ESLint config file in project root. Answer the questions as you like, but note that like Code Realm I chose JSON format for the config file.

```sh
npx eslint --init
```

3. Create a `src` directory in project root and make an `index.js` file inside of it. As an example of manual ESLint usage, the first line lints all JS files in the `src` directory, and the second does the same but also does as many automated fixes as it can:

```sh
npx eslint src/*.js
npx eslint src/*.js --fix
```

4. Install [prettier-eslint](https://github.com/prettier/prettier-eslint) as a development dependency along with prettier-eslint-cli:

```sh
npm i -D prettier-eslint prettier-eslint-cli
```

5. To print to standard output prettified JavaScript from all files in the `src` directory (even those in nested directories) via the CLI:

```sh
npx prettier-eslint 'src/**/*.js'
```

6. Same as above, but actually write the output to the JavaScript files instead of standard output:

```sh
npx prettier-eslint 'src/**/*.js' --write
```

7. Modify the scripts section of `package.json` so it has the following linting scripts:

```json
{
  "scripts": {
    "lint": "eslint '**/*.js'",
    "lint:fix": "prettier-eslint '**/*.js' --write"
  }
}
```

8. Note that [I'm not the only one who has had issues with the syntax used in #7](https://github.com/prettier/prettier-eslint-cli/issues/208). If you're on linux, the following alternative syntax might fix it:

```json
{
  "scripts": {
    "lint": "eslint $PWD/'**/*.js'",
    "lint:fix": "prettier-eslint $PWD/'**/*.js' --write"
  }
}
```

9. To manually use the scripts we just created:

```sh
npm run lint
npm run lint:fix
```

10. Create a configuration for VS Code to format code automatically on save. Create a `.vscode` directory in project root, and inside it make a `settings.json` file. Inside include (and restart vs code after):

```json
{
  "editor.formatOnSave": true,
  "prettier.eslintIntegration": true
}
```

11. Don't forget to make sure you have the ESLint and Prettier plugins for VS Code installed. Finally push your changes to GitHub!

---

## Husky and lint-staged

This next step is kind of optional. If the scope of your project is small enough that you know you are the only one who will ever work on it, and you'll always be editing it with VS Code... the steps we've already taken with Prettier and ESLint will probably be enough.

But if you're going to have multiple contributors to your repo, or if you're going to be editing it in VIM or some other editor from time to time... this section will automate the linting and formatting steps to occur on every git commit.

1. First install [Husky](https://github.com/typicode/husky).

```sh
npm i -D husky
```

2. Next install [lint-staged](https://github.com/okonet/lint-staged). There are newer instructions on GitHub, but for simplicity I'm just going to follow Code Realm's instructions which seem compatible.

```sh
npm i -D lint-staged
```

3. Include the following configuration in `package.json`:

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": ["npm run lint:fix", "git add"]
  }
}
```

4. Commit your changes and push to Github.

---

## Rollup

Instead of WebPack or Parcel, we'll use [Rollup](https://rollupjs.org/guide/en/). It seems to be more popular with those authoring Libraries as opposed to web applications.

1. Install dependencies:

```sh
npm i -D rollup rollup-plugin-node-resolve
```

2. In project root, create a rollup config file:

```sh
touch rollup.config.js
```

3. Here is a basic rollup config that will use `src/index.js` as entry point and build up three output bundles - cjs, esm, umd - and stick them in a `dist` directory:

```js
const prefix = "dist/bundle.";

export default {
  input: "src/index.js",
  output: [
    {
      file: `${prefix}cjs.js`,
      format: "cjs",
    },
    {
      file: `${prefix}esm.js`,
      format: "esm",
    },
    {
      name: "YourAwesomePackageNameHere",
      file: `${prefix}umd.js`,
      format: "umd",
    },
  ],
};
```

4. Install [rimraf](https://www.npmjs.com/package/rimraf) as a development dependency to clean out the dist directory every time rollup runs:

```sh
npm i -D rimraf
```

5. Add the scripts for `prebuild` and `build` to `package.json`:

```json
{
  "scripts": {
    "prebuild": "rimraf dist",
    "build": "rollup -c"
  }
}
```

6. Now you can commit your changes and push to GitHub. Every time you run the following, your `dist` directory will be cleaned out and you'll get three new bundles from rollup:

```sh
npm run build
```

---

## Babel... and Maybe React

So in [this video](https://www.youtube.com/watch?v=4joAZ2RQNys) Code Realm goes through explaining how to install React support via Babel. I am interested in Babel, but only for plain ES6+ features, not React.

I have nothing against React, but the whole point of this workflow for me is learning a modern vanilla JavaScript process. There are already tools like Create-React-App, and even [Create-React-Library](https://github.com/transitive-bullshit/create-react-library).

So I'm going to omit the React stuff, but feel free to go that route if you like.

1. Install Babel support:

```sh
npm i -D rollup-plugin-babel @babel/core @babel/preset-env
```

2. Create a `.babelrc` file in project root. Here's the config that Code Realm uses (Obviously, don't add `node` if you aren't targeting that environment. [Check out the docs](https://babeljs.io/docs/en/babel-preset-env#targets) to customize this part.):

```json
{
  "presets": [
    [
      "@babel/env",
      {
        "modules": false,
        "targets": {
          "browsers": "> 0.25%, ie 11, not op_mini all, not dead",
          "node": 10
        }
      }
    ]
  ]
}
```

3. Add the following two `import` statements to the top of the `rollup.config.js` file

```js
import babel from "rollup-plugin-babel";
import resolve from "rollup-plugin-node-resolve";
```

4. Also add the following `plugins` section to the default exported object in `rollup.config.js`:

```js
{
  plugins: [
    resolve(),
    babel({
      exclude: "node_modules/**",
    }),
  ],
}
```

5. If there are any **external** **dependencies** for your project, list their names as strings in an array called `external` in `rollup.config.js`. For this example, I'm including lodash. The point of this is so that your package indicates to consumers that the peer dependency, in this case lodash, is not itself bundled with the package but must be installed by the consumer:

```js
{
  external: ["lodash"];
}
```

6. For instances where you are generating a `umd` bundle like we are in this example, you need to specify any global keywords for the `umd` bundle in `rollup.config.js`, such as if you are using `React` or `jquery`:

```js
const prefix = "dist/bundle.";

export default {
  input: "src/index.js",
  output: [
    {
      file: `${prefix}cjs.js`,
      format: "cjs",
    },
    {
      file: `${prefix}esm.js`,
      format: "esm",
    },
    {
      name: "YourAwesomePackageNameHere",
      globals: {
        jquery: "$",
        react: "React",
        "react-dom": "ReactDOM",
      },
      file: `${prefix}umd.js`,
      format: "umd",
    },
  ],
};
```

7. At this point we can in project root create two files, `.prettierignore` and `.eslintignore`. Put the following in both of them to stop the plugins from messing with node modules and dist bundles. Also add the `dist` directory to `.gitignore` if you haven't already:

```
dist

node_modules
```

8. Code Realm at this point has us install the `Terser` plugin to help us minify our code.

```sh
npm i -D rollup-plugin-terser
```

9. Import the plugin into the `rollup.config.js` file, as well as create a constant right below it (not within default exported config object) that determines whether rollup is in production mode or not:

```js
import { terser } from "rollup-plugin-terser";

const production = !process.env.ROLLUP_WATCH;
```

10. Now add the terser plugin to the plugins array, making sure it only runs when in production mode:

```js
{
  plugins: [
    resolve(),
    babel({
      exclude: "node_modules/**",
    }),
    production && terser(),
  ];
}
```

11. Test your build process with `npm run build` if you like. Now might be a good time to make sure that the `main` field in `package.json` points to the `dist/bundle.cjs.js` file, as that is the format NPM will be expecting. Right below the main field, you can put the following two fields to enable users of, say, Webpack to utilize the `esm` bundle and enable tree-shaking in their project. Note that `sideEffects` being false essentially means it is safe to tree-shake the project and scrap the code in your bundle they aren't using. The example that Code Realm uses is that maybe your package provides multiple components for your users, and as long as your components don't depend on one another, it's safe for your users to have Webpack tree-shake the code and scrap the components they aren't using.

```json
{
  "main": "dist/bundle.cjs.js",
  "module": "dist/bundle.esm.js",
  "sideEffects": false
}
```

12. Finally, push your changes to GitHub.

---

## Component Pattern

Just for documentation purposes, I'd like to note the component pattern that Code Realm used in the 7th video of his playlist. It's very easy to understand, but I've never actually bothered to implement this pattern myself so should I ever want to I'll have an easy reference right here. He does it of course with React/JSX and some CSS stuff, but I'm just going to try to notate a Vanilla JavaScript version.

Say the `src/` folder has the following layout:

```sh
src/
    index.js
    component1/
        component1.js
        index.js
    component2/
        component2.js
        index.js
```

The `src/index.js` might look like the following:

```js
export * from './component1

export * from './component2
```

Each component subfolder itself has two files. The `component1/index.js` might look like the following:

```js
export { default as component1 } from "./component1";
```

So what might `component1/component1.js` look like?

```js
const component1 = (msg) => {
  return {
    result: `Your input: ${msg}`,
    retrieve: () => {
      console.log(this.result);
    },
  };
};

export default component1;
```

Nothing groundbreaking lol. But you get the idea.

---

## Notes on Example Projects

In project root, you can create an `examples` folder that contains example projects that use your bundles. Importantly, in the `package.json` file of each example project folder, you'll want to set `private` to `true` so the example projects stay on GitHub and don't end up on NPM. Also make sure to provide a helpful `README.md` file for each example project.

Although I've never tried it, I've read that installing your own project as a dependency in your example project can be done in several ways. The way that Code Realm does involves using `yarn add ../..` or `npm i ../..` in his example project roots. One potential alternative is using the `npm link` functionality. Simply run `npm link` in your module project's root folder on your development machine. Then, on the same machine but in the nested project directory, run `npm link <your-project-name>`, obviously with your module's name inserted at the end.

To create a `cdn/umd` bundle example before you actually distribute the project to NPM, you can simply `npm i ../..` or similar in your `cdn-example` root directory, and then you'd reference in a relative path such as the following in a script tag in an `index.html` file:

```html
<script src="./node_modules/my-awesome-module/dist/bundle.umd.js"></script>
<script>
  // usage of your module here
</script>
```

---

## Testing with Jest

[Obligatory link to the Jest docs](https://jestjs.io/docs/en/getting-started.html).

1. Install `jest` as a development dependency along with `babel-jest` compatibility package:

```sh
npm i -D jest babel-jest
```

2. To check for and run any tests from the command line:

```sh
npx jest
```

3. There are multiple ways to indicate in your project structure that there are test files for Jest. The convention I'll use is creating a new file in every directory with files that have module functionality and adding a `-.test.js` filename suffix. Example:

```sh
component1.js
component1.test.js
```

4. To get eslint to stop complaining about the syntax in test files, install `eslint-plugin-jest`. The following is how you then might modify the `.eslintrc.json` file:

```json
{
  "extends": ["standard", "plugin:jest/recommended"]
}
```

5. If you now try to run the tests it might fail. One thing you may need to do as add another entry called "env" to the json config object in `.babelrc`:

```json
{
  "presets": [...],
  "env": {
    "test": {
      "presets": ["@babel/env", "@babel/react"]
    }
  }
}
```

6. With the testing example Code Realm used, when he ran `npx jest`, it would output a snapshot file. In future runs of the command, it would compare the new output and fail the test. To get by this, in the future you can run `npx jest -u` to update the snapshot. This is mostly for writing UI components I think. Admittedly, at the time of this writing I have ZERO experience personally with Jest, so these are just my personal notes based on the tutorial I was watching.

¯\\\_(ツ)\_/¯

7. You can add the following four test-related scripts to your `package.json`. To manually generate a coverage report based on those tests, you can run `npm test -- --coverage`:

```json
{
  "test": "jest",
  "test:watch": "jest --watch",
  "test:coverage": "jest --coverage",
  "test:staged": "jest --findRelatedTests"
}
```

8. You might want to add the new `coverage` file to the `.gitignore`, `.eslintignore`, and `.prettierignore` files. Also, you can add the script `npm run test:staged` to the middle of the`lint-staged` part of your `package.json`:

```json
{
"lint-staged": {
  "*.js": [
    "npm run lint:fix",
    "npm run test:staged",
    "git add"
  ]
}
```

---

## TravisCI and Coveralls

DISCLAIMER: once again, although I've read about and heard of these services before, as of this writing I've yet to personally use them. These are just my personal notes based on the Code Realm's YouTube playlist.

1. Install the `node-coveralls` package as a preliminary development dependency. We'll get back to this later.

```sh
npm i -D coveralls
```

2. After activating your project repository in TravisCI's interface, create a `.travis.yml` file in your project root and put the following contents in it. Note that the `node` value under the `node_js` key indicates to TravisCI that you want them to install the latest version of node. If you want to install a specific version you can also indicate that.

```yml
sudo: false

language: node_js

node_js:
  - node

script:
  - npm run lint
  - npm run test:coveralls

notifications:
  email: false
```

3. In your `package.json` add the following to your scripts:

```json
{
  "scripts": {
    ...
    "test:coveralls": "jest --coverage --coverageReporters=text-lcov | coveralls",
    ...
  }
}
```

4. You can add the markdown badges from TravisCI and Coveralls to your project's README so you can look like a cool kid on GitHub.

(•\_•) ( •\_•)>⌐■-■ (⌐■_■)

---

## Publishing to NPM

1. Rather than making a file called `.npmignore` in your project root to **blacklist** certain project files from being published to NPM, you can **whitelist** them instead by creating a listing for the `dist` folder under a `files` key in your `package.json`:

```json
{
  "files": ["dist"]
}
```

2. Also create the following script in `package.json` to make sure your `dist` folder is up to date before publishing:

```json
{
  "scripts": {
    ...
    "prepublishOnly": "npm run build",
    ...
  }
}
```

3. After publishing, if you want to use the UMD bundle in a script tag, you can grab the URL for that version by replacing the `cjs` portion of the UNPKG URL with `umd`.
