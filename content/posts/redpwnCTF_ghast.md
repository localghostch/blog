---
title: "RedpwnCTF - Web - Ghast"
date: 2019-08-16
draft: false
---

RedpwnCTF 2019 – Crypt

**Category :** Web
**Description :** Store your most valuable secrets with this new encryption algorithm.

## Write-up

Source code of the server was given: 

Package.json: 

```json
{
  "name": "ghast",
  "version": "0.0.0",
  "private": true,
  "dependencies": {
    "cookie": "^0.4.0",
    "raw-body": "^2.4.1"
  }
}

 ```


 ghast.js:

 ```javascript
 const { promisify } = require('util')
const http = require('http')
const rawBody = promisify(require('raw-body'))
const cookie = require('cookie')
const secrets = require('./secrets')

let idIdx = 0

const makeId = () => Buffer.from(`ghast:${idIdx++}`).toString('base64').replace(/=/g, '')

const things = new Map()

things.set(makeId(), {
  name: secrets.adminName,
  // to prevent abuse, the admin account is locked
  locked: true,
})

const registerPage = `
<!doctype html>
<form id=form>
  your name: <br><input type=text id=uname><br><br>
  <button type=submit>submit</button>
</form>
<script>
  form.addEventListener('submit', async (evt) => {
    evt.preventDefault()
    const res = await fetch('/api/register', {
      method: 'POST',
      body: JSON.stringify({
        name: uname.value,
      }),
    })
    const text = await res.text()
    if (res.status === 200) {
      document.cookie = 'user=' + encodeURIComponent(text)
      location = '/ghasts/make'
    } else {
      alert(text)
    }
  })
</script>
`

const ghastMakePage = `
<!doctype html>
<form id=form>
  ghast name: <br><input type=text id=gname><br><br>
  ghast content: <br><textarea id=content></textarea><br><br>
  <button type=submit>submit</button>
</form>
<script>
  form.addEventListener('submit', async (evt) => {
    evt.preventDefault()
    const res = await fetch('/api/ghasts', {
      method: 'POST',
      body: JSON.stringify({
        name: gname.value,
        content: content.value,
      }),
    })
    const text = await res.text()
    if (res.status === 200) {
      location = '/ghasts/' + text
    } else {
      alert(text)
    }
  })
</script>
`

const ghastViewPage = `
<!doctype html>
<h1 id=gname></h1>
<div id=content></div>
<script>
  (async () => {
    const res = await fetch('/api/things/' + encodeURIComponent(location.pathname.replace('/ghasts/', '')))
    if (res.status === 200) {
      const body = await res.json()
      gname.textContent = body.name
      content.textContent = body.content
    } else {
      alert(await res.text())
    }
  })()
</script>
`

http.createServer(async (req, res) => {
  let user
  if (req.headers.cookie !== undefined) {
    const userId = cookie.parse(req.headers.cookie).user
    if (things.get(userId) === undefined && req.url !== '/register' && req.url !== '/api/register') {
      res.writeHead(302, {
        location: '/register',
      })
      res.end('')
      return
    } else {
      user = things.get(userId)
    }
  } else if (req.url !== '/register' && req.url !== '/api/register') {
    res.writeHead(302, {
      location: '/register',
    })
    res.end('')
    return
  }
  if (user !== undefined && (req.url === '/register' || req.url === '/')) {
    res.writeHead(302, {
      location: '/ghasts/make',
    })
    res.end('')
  }
  if (req.url === '/api/ghasts' && req.method === 'POST') {
    let body
    try {
      body = JSON.parse(await rawBody(req, {
        limit: '512kb',
      }))
      if (typeof body.name !== 'string' && typeof body.content !== 'string') {
        throw 1
      }
    } catch (e) {
      res.writeHead(400)
      res.end('bad body')
      return
    }
    const id = makeId()
    things.set(id, {
      name: body.name,
      content: body.content,
    })
    res.writeHead(200)
    res.end(id)
  } else if (req.url.startsWith('/api/things/') && req.method === 'GET') {
    const id = req.url.replace('/api/things/', '')
    if (things.get(id) === undefined) {
      res.writeHead(404)
      res.end('ghast not found')
    } else {
      res.writeHead(200)
      res.end(JSON.stringify(things.get(id)))
    }
  } else if (req.url === '/api/register' && req.method === 'POST') {
    let body
    try {
      body = JSON.parse(await rawBody(req, {
        limit: '512kb',
      }))
      if (typeof body.name !== 'string') {
        throw 1
      }
    } catch (e) {
      res.writeHead(400)
      res.end('bad body')
      return
    }
    if (body.name === secrets.adminName) {
      res.writeHead(403)
      res.end('no')
      return
    }
    const id = makeId()
    things.set(id, {
      name: body.name,
    })
    res.writeHead(200)
    res.end(id)
  } else if (req.url === '/api/flag' && req.method === 'GET') {
    if (user.locked) {
      res.writeHead(403)
      res.end('this account is locked')
      return
    }
    if (user.name === secrets.adminName) {
      res.writeHead(200)
      res.end(secrets.flag)
    } else {
      res.writeHead(403)
      res.end('only the admin can wield the flag')
    }
  } else if (req.url === '/register' && req.method === 'GET') {
    res.writeHead(200, {
      'content-type': 'text/html',
    })
    res.end(registerPage)
  } else if (req.url === '/ghasts/make' && req.method === 'GET') {
    res.writeHead(200, {
      'content-type': 'text/html',
    })
    res.end(ghastMakePage)
  } else if (req.url.startsWith('/ghasts/') && req.method === 'GET') {
    res.writeHead(200, {
      'content-type': 'text/html',
    })
    res.end(ghastViewPage)
  } else {
    res.writeHead(404)
    res.end('not found')
  }
}).listen(80, () => {
  console.log('listening on port 80')
})

```

As it is the first time we connect to the website, we are directly redirected to ```/api/register```
Name is not important because event if we find the admin name, we can't register as him.
We can learn from the code that when you register, the server create an ID for your account and set it in your cookies under name: ```user```


This part of code is really interesting: 

```javascript
let idIdx = 0

const makeId = () => Buffer.from(`ghast:${idIdx++}`).toString('base64').replace(/=/g, '')

const things = new Map()

things.set(makeId(), {
  name: secrets.adminName,
  // to prevent abuse, the admin account is locked
  locked: true,
})

```

We can learn from it that: 
1. id index  is set to 0 at start
2. id is ```ghast:${idIdx++}``` in base 64 (without ==)
3. Admin acount is created with index 0 but it is locked 


The first idea was to change the user cookie by the one from the admin: ```Z2hhc3Q6MA```
Sadly, a lock account can't get the flag from ```/api/flag```


Okay then let's look at the ghast we can create via ```/api/ghast```:
```javascript
  if (req.url === '/api/ghasts' && req.method === 'POST') {
    let body
    try {
      body = JSON.parse(await rawBody(req, {
        limit: '512kb',
      }))
      if (typeof body.name !== 'string' && typeof body.content !== 'string') {
        throw 1
      }
    } catch (e) {
      res.writeHead(400)
      res.end('bad body')
      return
    }
    const id = makeId()
    things.set(id, {
      name: body.name,
      content: body.content,
    })
    res.writeHead(200)
    res.end(id)
  }
``` 

We can see that ghasts are stored in the same map as users and with a common property : "name" 

Let's try to search for a ghast with the id of admin acount: ```/ghasts/Z2hhc3Q6MA```

Bingo, we have the name of the admin:  sherlockholmes99

As seen before, we can't register with the admin name because of this condition: 

```javascript
 if (body.name === secrets.adminName) {
      res.writeHead(403)
      res.end('no')
      return
    }
```

But we can create a ghast with this name.It will store a new ghast with same name as admin but without ```locked``` property and so it will bypass the ```/api/flag``` protections.
Last step is to but the ghast id in our user cookie.

Flag: flag{th3_AdM1n_ne3dS_A_n3W_nAme}






