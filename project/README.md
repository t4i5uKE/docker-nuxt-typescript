# project

> My awe-inspiring Nuxt.js project

# Introduction
[Nuxt TypeScript](https://typescript.nuxtjs.org/) has published the setup procedure, but since there was little information about building with docker-compose, I summarized it.

Also, since the introduction procedure to TypeScript has changed since Ver.2.9, and there were some personally confused parts, I would like to share based on that point.

# 1. Execution environment
* macOS Catalina Ver.10.15.1
* Docker version 19.03.4, build 9013bf5
* docker-compose version 1.24.1, build 4667896b
* Nuxt.js Ver.2.10.2
* node.js 12.13.0-alpine

# 2. Prerequisites
* The version of Nuxt.js is 2.9 or higher.
* Docker, docker-compose and node.js are installed on the PC.

If docker, docker-compose, and node.js are not installed, execute the following.

## 2.1. Install Docker
To install Docker, access [DockerHub](https://hub.docker.com/editions/community/docker-ce-desktop-mac) and download `Docker.dmg`.

* If you use DockerHub for the first time, you need to create an account.

When you start `Docker.dmg`, Docker will be installed. Copy Docker to Applications after the installation is completed.

Alternatively, it can be installed using a package management system called [Homebrew](https://brew.sh/index_en). The installation procedure is shown below.

```terminal
# Install Homebrew
$ /usr/bin/ruby ​​-e "$ (curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# Make sure Homebrew is installed correctly
$ brew doctor

# Install Docker
$ brew install docker
$ brew cask install docker
```

You can check the version of docker and Homebrew by executing the following command.

```terminal
# Docker
$ docker --version

Docker version 19.03.4, build 9013bf5

# Homebrew
$ brew --version

Homebrew 2.1.16
Homebrew/homebrew-core (git revision 00c2c; last commit XXXX-XX-XX)
Homebrew/homebrew-cask (git revision 9e283; last commit XXXX-XX-XX)
```

## 2.2. Install docker-compose
To install docker-compose, execute the following command.

```terminal
$ curl -L https://github.com/docker/compose/releases/download/1.3.1/docker-compose-`uname -s`-`uname -m`> /usr/local/bin/docker-compose

# Set permissions so that docker-compose command can be executed.
$ chmod + x /usr/local/bin/docker-compose
```

Note: If you get a `Permission denied` error during execution, the ` /usr/local/bin` directory may not be writable. At that time, it is necessary to install Compose as a superuser. Run `sudo -i`, then execute the above two commands and` exit`.

# 3. Nuxt.js environment construction
Before converting to TypeScript, it is necessary to create a Nuxt.js project first.

## 3.1. File structure
The file structure of this time is as follows. To avoid confusion of different files in one directory, create a new project directory and create a `Dockerfile` in it.

I also wanted to clean up the file hierarchy in the Nuxt.js project, so I created a new `src` directory and put it in it.

The file structure is shown below.

```terminal
.
├── README.md
├── docker-compose.yml
└── project
    ├── Dockerfile
    ├── README.md
    ├── node_modules
    ├── nuxt.config.ts
    ├── package.json
    ├── src
    │   ├── assets
    │   ├── components
    │   ├── layouts
    │   ├── middleware
    │   ├── pages
    │   ├── plugins
    │   ├── static
    │   ├── store
    │   └── vue-shim.d.ts
    ├── tsconfig.json
    ├── yarn-error.log
    └── yarn.lock
```

## 3.2. Dockerfile
First, create a Dockerfile. For `node.js`, the image is published on [DockerHub](https://hub.docker.com/_/node/), so use it.

```docker: Dockerfile

# Specify image
FROM node: 12.13.0-alpine

# Execute command
RUN apk update && \
    apk add git && \
    npm install -g @ vue / cli nuxt create-nuxt-app && \

```

This time, we used `alpine` to reduce the weight of the image.

* **alpine** : Linux distribution based on `BusyBox` and `musl` called Alpine Linux.

## 3.3. Docker-compose.yml
Next, create docker-compose.yml.

```docker: docker-compose.yml
version: '3'
services:
  node:
    # Dockerfile location
    build:
        context: ./
        dockerfile: ./project/Dockerfile
    working_dir: /home/node/app/project
    # Share source code in host and container
    volumes:
      -./:/home/node/app
    # Access 3000 inside container with 5000 from outside
    ports:
      -5000: 3000
    environment:
      -HOST = 0.0.0.0
```

## 3.4. Create docker image
After creating Dockerfile, docker-compose.yml, execute the following command to create a docker image.

```terminal
$ docker-compose build
```

After execution, you can check if the image is ready with the following command.

```terminal
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
docker-nuxt-typescript_node latest 7f8324973b48 3 days ago 434MB
node 12.13.0-alpine 5d187500daae 3 weeks ago 80.1MB
```

## 3.5. Launch Nuxt.js
If you want to execute `npm` and` yarn` commands in the container, specify as follows.

```terminal
$ docker-compose run --rm <Container Name> <Command>
```

Execute the following command to start Nuxt.js and set up.

```terminal
$ docker-compose run --rm nuxt npx create-nuxt-app ./project
Project name (your-title)
Project description (My wicked Nuxt.js project)
Author name (author-name)
Choose the package manager
> Yarn
  Npm
Choose UI framework (Use arrow keys)
> None
  Ant Design Vue
  Bootstrap Vue
  Buefy
  Bulma
  Element
  Framevuerk
  iView
  Tachyons
  Tailwind CSS
  Vuetify.js
Choose custom server framework (Use arrow keys)
> None (Recommended)
  AdonisJs
  Express
  Fastify
  Feathers
  hapi
  Koa
  Micro
# You can select by space. (Note that it is not the Enter key)
Choose Nuxt.js modules
> (*) Axios
 () Progressive Web App (PWA) Support
Choose linting tools
> (*) ESLint
 () Prettier
 () Lint staged files
Choose test framework
> None
  Jest
  AVA
Choose rendering mode (Use arrow keys)
  Universal (SSR) // WEB SITE
> Single Page App // WEB APPLICATION
Choose development tools
> (*) jsconfig.json (Recommended for VS Code)
```

The above settings are the ones I selected for setup. About setting, it is good to introduce what is necessary for each.

## 3.6. Container startup
Execute the following command to start the container.

```terminal
$ docker-compose run node npm run dev
# OR
$ docker-compose run node yarn run dev
```

After starting, access http://localhost:5000/ and once you can see the sample app screen, it is complete.

# 4. TypeScript
Finally, the procedure for changing Nuxt.js to TypeScript is explained.

## 4.1. Install @ nuxt/typescript-build, @ nuxt/typescript-runtime
Before proceeding with TypeScript, first install `@ nuxt/typescript-build` and` @nuxt/typescript-runtime` by executing the following commands.
(It works with either one, but install both this time)

* **@nuxt/typescript-build**: For handling TypeScript in `nuxt build`.
* **@nuxt/typescript-runtime**: For processing TypeScript under Node.js environment.

```terminal
$ docker-compose run node npm install --save-dev @ nuxt/typescript-build
$ docker-compose run node npm install @ nuxt/typescript-runtime
# OR
$ docker-compose run node yarn add --dev @ nuxt/typescript-build
$ docker-compose run node yarn add @ nuxt/typescript-runtime
```

Regarding `@nuxt/types`, it is included in the above two, so there is no need to install it.

## 4.2. Create tsconfig.json
Delete `jsconfig.json` created by Nuxt.js and create a new` tsconfig.json`.
The source of `tsconfig.json` is as follows.

```json:tsconfig.json
{
  "include": [
    "./src/**/*"
  ],
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "lib": [
      "esnext",
      "esnext.asynciterable",
      "dom"
    ],
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "allowJs": true,
    "sourceMap": true,
    "strict": true,
    "noEmit": true,
    "baseUrl": ".",
    "paths": {
      "~/*": [
        "./src/*"
      ],
      "@/*": [
        "./src/*"
      ]
    },
    "types": [
      "@types/node",
      "@nuxt/types",
      "@nuxtjs/axios"
    ]
  },
  "exclude": [
    "node_modules"
  ]
}
```

This time, the directory in the project is moved to the `src` directory, so the settings of` include`, `paths`, etc. are changed.

## 4.3. Create nuxt.config.ts
Next, delete `nuxt.config.js` and create a new` nuxt.config.ts`.
The source of `nuxt.config.ts` is as follows.

```ts:nuxt.config.ts
import { Configuration } from '@nuxt/types'
 
const nuxtConfig: Configuration = {
  mode: 'universal',
  buildModules: ['@nuxt/typescript-build'],
  server: {
    port: 5000,
    host: 'localhost',
  },
  head: {
    title: process.env.npm_package_name || '',
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: process.env.npm_package_description || '' }
    ],
    link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    ]
  },
  loading: { color: '#fff' },
  css: [
  ],
  plugins: [
  ],
  typescript: {
    typeCheck: true,
    ignoreNotFoundWarnings: true
  },
  modules: [
    '@nuxtjs/axios',
    '@nuxtjs/eslint-module',
  ],
  build: {
    extend(config, ctx) {
    }
  },
  srcDir: 'src/'
}
module.exports = nuxtConfig
```

From Ver.2.9, since the type definition of Configuration in `nuxt.config.ts` is exported from`@nuxt/types`, it is imported first.
Also, `@nuxt/typescript-build` is specified in the added [buildModules](https://nuxtjs.org/api/configuration-modules#-code-buildmodules-code-). With the change to `buildModules`, the` module` used in `build` can be specified in` devDependencies`.

## 4.4.Create vue-shim.d.ts
`vue-shim.d.ts` is a file required to recognize the described code as TypeScript. However, there is no need to import into the `.vue` file, and there is no problem if it is in the`src` directory.
The source of `vue-shim.d.ts` is as follows.

```ts:vue-shim.d.ts
declare module "*.vue" {
  import Vue from "vue";
  export default Vue;
}
```

## 4.5. Modification of package.json
`package.json` is created when Nuxt.js is set up, but since TypeScript is not applied, that point is corrected. The part to be modified is that `scripts` in`package.json` is a nuxt command, so change it to a `nuxt-ts` command.
The revised source is as follows.

```json:package.json
# Partially omitted
{
  # ...
  "scripts": {
    "dev": "nuxt-ts",
    "build": "nuxt-ts build",
    "start": "nuxt-ts start",
    "generate": "nuxt-ts generate",
    "lint": "eslint --ext .js,.vue --ignore-path .gitignore ."
  }
  # ...
}
```

## 4.6. Executing Nuxt TypeScript
Start Nuxt.js again based on the changes.

```terminal
$ docker-compose run node yarn run dev
yarn run v1.19.1
$ nuxt-ts

 ╭─────────────────────────────────────────────╮
 │                                             │
 │   Nuxt.js v2.10.2                           │
 │   Running in development mode (universal)   │
 │                                             │
 │   Listening on: http://localhost:5000/      │
 │                                             │
 ╰─────────────────────────────────────────────╯

ℹ Preparing project for development
ℹ Initial build may take a while
✔ Builder initialized
✔ Nuxt files generated
ℹ Starting type checking service...
ℹ Using 1 worker with 2048MB memory limit

✔ Client
  Compiled successfully in 13.16s

✔ Server
  Compiled successfully in 11.98s

ℹ No type errors found
ℹ Version: typescript 3.6.4
ℹ Time: 11648ms
ℹ Waiting for file changes
ℹ Memory usage: 254 MB (RSS: 315 MB)
```

If you look at the above, you can see that it is running with the `nuxt-ts` command.
After execution, access http://localhost:5000/ as before and confirm that the screen is displayed.

This completes TypeScript conversion.

# 5. Summary
This time, I built Nuxt TypeScript with docker-comnpose. It would be greatly appreciated if the changes from Ver.2.9 could support people who stopped working under docker-compose.

In addition, since this sample is published on [GitHub](https://github.com/t4i5uKE/docker-nuxt-typescript), if there are any unclear points or correction points, I will try to correspond at any time .
