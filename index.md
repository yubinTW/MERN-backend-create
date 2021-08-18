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
    max-height: 65vh;
}
</style>

---

Yubin, Hsu

TSID / NTAP

ybhsu@tsmc.com

---

### Outline

- Create the backend project
- MongoDB support
- Implement the API
- Define API routes as a plugin
- Define response schema

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

---

create ```.gitignore```

```plaintext=
node_module
out
```

--

get more .gitignore example

https://github.com/github/gitignore

---

Add mongo support

--

Start a mongodb server

- using docker

```
docker run -d -p 27017:27017 mongo
```

- or binary

```
./mongod.exe
```

--

Install mongoose

```
npm i mongoose
npm i -D @types/mongoose
```

--

Add env variables in backend/.env

```
MONGO_HOST=localhost
MONGO_PORT=27017
MONGO_DATABASE=myMERN
```

--

Create backend/src/plugins/mongoose.ts

```typescript=
import mongoose from 'mongoose'

const host = process.env.MONGO_HOST || 'localhost'
const port = process.env.MONGO_PORT || 27017
const database = process.env.MONGO_DATABASE || 'fastify'

const establishConnection = () => {
  if (!process.env.JEST_WORKER_ID && mongoose.connection.readyState === 0) {
    mongoose.connect(
      `mongodb://${host}:${port}/${database}`,
      { useNewUrlParser: true, useUnifiedTopology: true, useFindAndModify: false },
      (err) => {
        if (!err) console.log('MongoDB connection successful.')
        else console.log('Error in DB connection : ' + JSON.stringify(err, undefined, 2))
      }
    )
  }
}

export { establishConnection }
```

--

Edit backend/src/server.ts

```typescript=
import { establishConnection } from './plugins/mongoose'

// ...

  server.listen(port, (err, _) => {
      if (err) {
          console.error(err)
      }
      establishConnection()
  })

// ...
```

--

Start server

![](res/2021-08-17-03-19-36.png)

```MongoDB connection successful```

---

Add mongo schema

https://mongoosejs.com/docs/typescript.html

--

Define interface

create backend/src/types/cat.ts

```typescript=
interface ICat {
    name: string
    weight: number
}

export { ICat }
```

--

Add mongo schema

create backend/src/models/cat.ts

```typescript=
import { model, Schema } from 'mongoose'
import { ICat } from '../types/cat'

const catSchema: Schema = new Schema(
    {
        name: {
            type: String,
            required: true
        },
        weight: {
            type: Number,
            default: 0
        }
    },
    {
        timestamps: true
    }
)

export default model<ICat>('Cat', catSchema)
```

---

Get Cats

--

Add Cat Repo

create backend/src/repo/cat-repo

```typescript=
import { ICat } from './../types/cat'
import Cat from './../models/cat'

interface CatRepo {
  getCats(): Promise<Array<ICat>>
}

class CatRepoImpl implements CatRepo {
  private constructor() {}

  static of(): CatRepoImpl {
    return new CatRepoImpl()
  }

  async getCats(): Promise<Array<ICat>> {
    return Cat.find()
  }

}

export { CatRepoImpl }
```

--

Add API endpoint

--

edit backend/src/server.ts

- GET /cats

```typescript=
server.get('/cats', async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const cats = await catRepo.getCats()
    return reply.status(200).send({ cats })
  } catch (error) {
    return reply.status(500).send({ msg: 'Internal Server Error' })
  }
})
```

--

- send GET request by API Client

![](res/2021-08-17-04-53-04.png)

---

Add Cat

--

edit backend/src/repo/cat-repo.ts

```typescript=
import { ICat } from './../types/cat'
import Cat from './../models/cat'

interface CatRepo {
  getCats(): Promise<Array<ICat>>
  addCat(catBody: ICat): Promise<ICat>
}

class CatRepoImpl implements CatRepo {
  private constructor() {}

  static of(): CatRepoImpl {
    return new CatRepoImpl()
  }

  async getCats(): Promise<Array<ICat>> {
    return Cat.find()
  }

  async addCat(catBody: ICat): Promise<ICat> {
    return Cat.create(catBody)
  }

}

export { CatRepoImpl }
```

--

Add API endpoint

--

edit backend/src/server.ts

- POST /cats

```
server.post('/cats', async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const catBody = request.body as ICat
    const cat = await catRepo.addCat(catBody)
    return reply.status(201).send({ cat })
  } catch (error) {
    return reply.status(500).send({ msg: 'Internal Server Error' })
  }
})
```

--

Send POST request by API Client

![](res/2021-08-17-05-06-19.png)

--

Get /cats again

![](res/2021-08-17-05-09-43.png)

---

Rebuild automatically when the code is changed

--

- install concurrently and nodemon

```
npm i -D concurrently nodemon
```

- add the script in package.json

```planttext
"dev": "concurrently \"tsc -w \" \"nodemon out/index.js\""
```

- run the script

```
npm run dev
```

---

prettier

--

install prettier

```
npm i -D prettier
```

--

add backend/.prettierrc

```planttext
{
  "semi": false,
  "singleQuote": true,
  "printWidth": 120,
  "trailingComma": "none",
  "arrowParens": "always"
}
```

ref: https://prettier.io/docs/en/options.html

--

- add the script in package.json

```planttext
"fix-prettier": "prettier --write \"./{src,test,examples,scripts}/**/*.ts\""
```

- format code by prettier

```
npm run fix-prettier
```

---

Define API routes as a plugin

ref: https://www.fastify.io/docs/master/Plugins/#create-a-plugin

--

create backend/src/routes/cat.ts

move API from server.ts to this file

```typescript=
import { FastifyInstance, RouteShorthandOptions, FastifyReply } from 'fastify'
import { ICat } from '../types/cat'
import { CatRepoImpl } from './../repo/cat-repo'

const CatRouter = (server: FastifyInstance, opts: RouteShorthandOptions, done: (error?: Error) => void) => {

  server.get('/cats', async (request, reply) => {
    const catRepo = CatRepoImpl.of()
    try {
      const cats = await catRepo.getCats()
      return reply.status(200).send({ cats })
    } catch (error) {
      return reply.status(500).send({ msg: 'Internal Server Error' })
    }
  })

  server.post('/cats', async (request, reply) => {
    const catRepo = CatRepoImpl.of()
    try {
      const catBody = request.body as ICat
      const cat = await catRepo.addCat(catBody)
      return reply.status(201).send({ cat })
    } catch (error) {
      return reply.status(500).send({ msg: 'Internal Server Error' })
    }
  })

  done()
}

export { CatRouter }

```

--

register CatRouter

edit backend/src/server.ts

```typescript=
server.register(CatRouter, { prefix: '/v1' })
```

--

![](res/2021-08-17-05-36-46.png)

---

DIY: implement update and delete API

- https://www.fastify.io/docs/v1.14.x/TypeScript/
- https://mongoosejs.com/docs/guide.html

---


url parameter

--

PUT /cats/:id

--

```typescript=
server.put('/cats/:id', async (request, reply) => {
  const id = request.params.id
  // ...
})
```

![](res/2021-08-17-07-27-26.png)

--

```typescript=
interface IdParams {
    id: string
}
server.put<{ Params: IdParams }>('/cats/:id', async (request, reply) => {
  const id = request.params.id
  // ...
})
```

ref: https://www.fastify.io/docs/v1.14.x/TypeScript/

---

Update Cat

--

edit backend/src/repo/cat-repo.ts

```typescript=
async updateCat(id: String, catBody: ICat): Promise<ICat | null> {
  return Cat.findByIdAndUpdate(id, catBody, { new: true })
}
```

ref: https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate

--

Add API endpoint

PUT /cats/:id

```typescript=
server.put<{ Params: IdParams; Body: ICat }>('/cats/:id', async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const catBody = request.body
    const id = request.params.id
    const cat = await catRepo.updateCat(id, catBody)
    return reply.status(200).send({ cat })
  } catch (error) {
    return reply.status(500).send({ msg: error })
  }
})
```

--

if id not found

```typescript=
const cat = await catRepo.updateCat(id, catBody)
if (cat) {
    return reply.status(200).send({ cat })
} else {
    return reply.status(404).send({msg: `Cat #${id} Not Found`})
}
```

--

if id is invalid

```typescript=
import { Types } from 'mongoose'

const id = request.params.id
if (!Types.ObjectId.isValid(id)) {
    return reply.status(400).send({msg: `Invalid id`})
}
```

--

```typescript=
server.put<{ Params: IdParams; Body: ICat }>('/cats/:id', async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const catBody = request.body
    const id = request.params.id
    if (!Types.ObjectId.isValid(id)) {
        return reply.status(400).send({msg: `Invalid id`})
    }
    const cat = await catRepo.updateCat(id, catBody)
    if (cat) {
        return reply.status(200).send({ cat })
    } else {
        return reply.status(404).send({msg: `Cat #${id} Not Found`})
    }
  } catch (error) {
    return reply.status(500).send({ msg: error })
  }
})
```

---

Delete Cat

--

edit backend/src/repo/cat-repo.ts

```typescript=
async deleteCat(id: string): Promise<ICat | null> {
  return Cat.findByIdAndDelete(id)
}
```

ref: https://mongoosejs.com/docs/api.html#model_Model.findByIdAndDelete

--

Add API endpoint

DELETE /cats/:id

```typescript=
server.delete<{ Params: IdParams }>('/cats/:id', async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const id = request.params.id
    if (!Types.ObjectId.isValid(id)) {
      return reply.status(400).send({ msg: `Invalid id` })
    }
    const cat = await catRepo.deleteCat(id)
    if (cat) {
      return reply.status(204).send()
    } else {
      return reply.status(404).send({ msg: `Cat #${id} Not Found` })
    }
  } catch (error) {
    return reply.status(500).send({ msg: error })
  }
})
```

---

DIY: Implement get one cat by id

- GET /cats/:id

---

Define response schema

--

typebox

```
npm i @sinclair/typebox
```

ref: https://github.com/sinclairzx81/typebox

--

edit backend/src/routes/cat.ts

```typescript=
import { Type, Static } from '@sinclair/typebox'
// ...
// in CatRouter
const CatsResponse = {
  cats: Type.Array(
    Type.Object({
      _id: Type.String(),
      name: Type.String(),
      weight: Type.Number()
    })
  )
}
type CatsResponse = Static<typeof CatsResponse>
opts = { ...opts, schema: { response: { 200: CatsResponse } } }

// put opts as the second parameter
server.get('/cats', opts, async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const cats = await catRepo.getCats()
    return reply.status(200).send({ cats })
  } catch (error) {
    return reply.status(500).send({ msg: 'Internal Server Error' })
  }
})
```

--

Get the response schema as defined

![](res/2021-08-17-06-54-06.png)

--

difference schemas based on difference status code

```typescript=
const CatsResponse = {
  cats: Type.Array(
    Type.Object({
      _id: Type.String(),
      name: Type.String(),
      weight: Type.Number()
    })
  )
}
type CatsResponse = Static<typeof CatsResponse>

const CatResponse = {
  cat: Type.Object({
    _id: Type.String(),
    name: Type.String(),
    weight: Type.Number()
  })
}
type CatResponse = Static<typeof CatResponse>

opts = { ...opts, schema: { response: { 200: CatsResponse, 201: CatResponse } } }

server.get('/cats', opts, async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const cats = await catRepo.getCats()
    return reply.status(200).send({ cats })
  } catch (error) {
    return reply.status(500).send({ msg: 'Internal Server Error' })
  }
})

server.post('/cats', opts, async (request, reply) => {
  const catRepo = CatRepoImpl.of()
  try {
    const catBody = request.body as ICat
    const cat = await catRepo.addCat(catBody)
    return reply.status(201).send({ cat })
  } catch (error) {
    return reply.status(500).send({ msg: 'Internal Server Error' })
  }
})
```

---

end