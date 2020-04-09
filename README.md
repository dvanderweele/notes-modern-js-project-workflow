# Notes on Modern JavaScript Project Workflow
These notes are derived from the Code Realm YouTube channel's React / JS Project Lifecycle playlist; I have no affiliation with the channel. These notes are more focused on the vanilla JavaScript aspects of the workflow, but for the sake of completeness I suppose some of the React-related notes will be included. I'm not focused on recreating the React CSS Spinners project he did, but instead I'm just documenting the overall process.

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
3) Create a `src` directory in project root and make an `index.js` file inside of it. As an example of manual ESLint usage, the lint all JS files in the `src` directory:
```sh
npx eslint src/*.js
```