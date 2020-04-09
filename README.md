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

1) List existing remotes
```git
$ git remote -v
> origin  https://github.com/USERNAME/REPOSITORY.git (fetch)
> origin  https://github.com/USERNAME/REPOSITORY.git (push)
```
2) Change remote's URL from HTTPS to SSH
```git
$ git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```
3) Check to see if you did it correctly
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

1) Setup ESLint as a development dependency in the project.
```sh
npm install eslint --save-dev
```
**or**
```sh
npm i -D eslint
```
2) Initialize the ESLint config file in project root. Answer the questions as you like, but note that like Code Realm I chose JSON format for the config file.
```sh
npx eslint --init
```
3) Create a `src` directory in project root and make an `index.js` file inside of it. As an example of manual ESLint usage, the first line lints all JS files in the `src` directory, and the second does the same but also does as many automated fixes as it can:
```sh
npx eslint src/*.js
npx eslint src/*.js --fix
```
4) Install [prettier-eslint](https://github.com/prettier/prettier-eslint) as a development dependency along with prettier-eslint-cli:
```sh
npm i -D prettier-eslint prettier-eslint-cli
```
5) To print to standard output prettified JavaScript from all files in the `src` directory (even those in nested directories) via the CLI:
```sh
npx prettier-eslint 'src/**/*.js'
```
6) Same as above, but actually write the output to the JavaScript files instead of standard output:
```sh
npx prettier-eslint 'src/**/*.js' --write
```
7) Modify the scripts section of `package.json` so it has the following linting scripts:
```json
{
  "scripts": {
    "lint": "eslint '**/*.js'",
    "lint:fix": "prettier-eslint '**/*.js' --write"
  }
}
```
8) Note that [I'm not the only one who has had issues with the syntax used in #7](https://github.com/prettier/prettier-eslint-cli/issues/208). If you're on linux, the following alternative syntax might fix it:
```json
{
  "scripts": {
    "lint": "eslint $PWD/'**/*.js'",
    "lint:fix": "prettier-eslint $PWD/'**/*.js' --write"
  }
}
```
9) To manually use the scripts we just created:
```sh
npm run lint
npm run lint:fix
```
10) Create a configuration for VS Code to format code automatically on save. Create a `.vscode` directory in project root, and inside it make a `settings.json` file. Inside include (and restart vs code after):
```json
{
  "editor.formatOnSave": true,
  "prettier.eslintIntegration": true
}
```
11) Don't forget to make sure you have the ESLint and Prettier plugins for VS Code installed. Finally push your changes to GitHub!

---
## Husky and lint-staged

This next step is kind of optional. If the scope of your project is small enough that you know you are the only one who will ever work on it, and you'll always be editing it with VS Code... the steps we've already taken with Prettier and ESLint will probably be enough.

But if you're going to have multiple contributors to your repo, or if you're going to be editing it in VIM or some other editor from time to time... this section will automate the linting and formatting steps to occur on every git commit.

1) First install [Husky](https://github.com/typicode/husky).
```sh
npm i -D husky
```
2) Next install [lint-staged](https://github.com/okonet/lint-staged). There are newer instructions on GitHub, but for simplicity I'm just going to follow Code Realm's instructions which seem compatible.
```sh
npm i -D lint-staged
```
3) Include the following configuration in `package.json`:
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": [
      "npm run lint:fix",
      "git add"
    ]
  },
}
```
4) Commit your changes and push to Github.

---
## Rollup

Instead of WebPack or Parcel, we'll use [Rollup](https://rollupjs.org/guide/en/). It seems to be more popular with those authoring Libraries as opposed to web applications.

1) Install dependencies:
```sh
npm i -D rollup rollup-plugin-node-resolve
```
2) In project root, create a rollup config file:
```sh
touch rollup.config.js
```
3) Here is a basic rollup config that will use `src/index.js` as entry point and build up three output bundles - cjs, esm, umd - and stick them in a `dist` directory:
```js
const prefix = 'dist/bundle.'

export default {
  input: 'src/index.js',
  output: [
    {
      file: `${prefix}cjs.js`,
      format: 'cjs'
    },
    {
      file: `${prefix}esm.js`,
      format: 'esm'
    },
    {
      name: 'YourAwesomePackageNameHere',
      file: `${prefix}umd.js`,
      format: 'umd'
    }
  ]
}
```
4) Install [rimraf](https://www.npmjs.com/package/rimraf) as a development dependency to clean out the dist directory every time rollup runs:
```sh
npm i -D rimraf
```
5) Add the scripts for `prebuild` and `build` to `package.json`:
```json
{
  "scripts": {
    "prebuild": "rimraf dist",
    "build": "rollup -c"
  }
}
```
6) Now you can commit your changes and push to GitHub. Every time you run the following, your `dist` directory will be cleaned out and you'll get three new bundles from rollup:
```sh
npm run build
```

---
## Babel... and Maybe React

