---

theme : "night"
transition: "slide"
highlightTheme: "monokai"
# logoImg: "logo.png"
slideNumber: true
title: "MERN Backend"

---

### MERN Backend Practice

<style>
pre {
  background: #303030;
  padding: 10px 16px;
  border-radius: 0.3em;
  counter-reset: line;
}
pre code[class*="="] .line {
  display: block;
  line-height: 1.8rem;
  font-size: 1em;
}
pre code[class*="="] .line:before {
  counter-increment: line;
  content: counter(line);
  display: inline-block;
  border-right: 3px solid #6ce26c !important;
  padding: 0 .5em;
  margin-right: .5em;
  color: #afafaf !important;
  width: 24px;
  text-align: right;
}

.reveal .slides > section > section {
  text-align:left; 
}

h1,h2,h3,h4 {
  text-align: center
}

p {
  text-align: center;
}

.present img {
    height: 65vh;
}
</style>

---

Yubin, Hsu

TSID / NTAP

ybhsu@tsmc.com

---

### Outline

- Create the backend project 

---

### Create the backend project

--

- create project folder

```bash=
mkdir myMernProject
cd myMernProject
```

- create backend project

```bash=
mkdir backend
cd backend
npm init
```

--

- Install dependencies of backend project

```bash=
npm i fastify pino-pretty dotenv
npm i -D typescript @types/node
npx tsc --init
```

--

edit backend/tsconfig.json

```json=
"include": ["src/**/*.ts"],
"exclude": ["node_modules"],

"compilerOptions": {
  "outDir": "./out",
  "rootDir": "./src",
}
```

--

create backend/.env

```plaintext
FASTIFY_PORT=8888
FASTIFY_ENABLE_LOGGING=true
ENV=dev
```

--

create backend/src/server.ts

```typescript=
import fastify, { FastifyInstance, FastifyReply, FastifyRequest } from 'fastify'
import { Server, IncomingMessage, ServerResponse } from 'http'

const server: FastifyInstance<Server, IncomingMessage, ServerResponse> = fastify({
    logger: { prettyPrint: true }
})

const startFastify: (port: number) => FastifyInstance<Server, IncomingMessage, ServerResponse> = (port) => {

    server.listen(port, (err, _) => {
        if (err) {
            console.error(err)
        }
    })

    server.get('/ping', async (request: FastifyRequest, reply: FastifyReply) => {
        return reply.status(200).send({ msg: 'pong' })
    })

    return server
}

export { startFastify }
```

--

create backend/src/index.ts

```typescript=
import { startFastify } from './server'
import * as dotenv from 'dotenv'

dotenv.config()
const port = process.env.FASTIFY_PORT || 8888

// Start your server
const server = startFastify(Number(port))

export { server }
```

--

#### build and run

- build with tsc

```
tsc
```

- run with node

```
node out/index.js
```

---

visit the endpoint

![](res/2021-08-13-05-10-53.png)

---

API client
- Postman
- curl
- Thunder Client (vscode extension)

--

Thunder Client

![](res/2021-08-13-05-12-51.png)

--

Send request by Thunder Client

![](res/2021-08-13-05-12-04.png)

--

Define custom npm scripts

- edit backend/package.json

```json=
"scripts": {
  "build": "tsc",
  "start": "node out/index.js"
},
```

--

Use custom npm script

```plaintext=
npm run build
npm run start
```

---

create ```.gitignore```

```plaintext=
node_module
out
```

--

get more example

https://github.com/github/gitignore
