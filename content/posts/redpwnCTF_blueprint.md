---
title: "RedpwnCTF - Web - Blueprint"
date: 2019-08-16
draft: false
---

 RedpwnCTF 2019 – Blueprint

**Category :** Web
**Description :** All the haxors are using blueprint. You created a blueprint with the flag in it, but the military-grade security of blueprint won't let you get it!

## Write-up

Source code of the server was given: 

Package.json: 

```json
{
  "name": "blueprint",
  "version": "0.0.0",
  "private": true,
  "dependencies": {
    "lodash": "4.17.11",
    "mustache": "^3.0.1",
    "raw-body": "^2.4.1"
  }
}
 ```


 blueprint.js: 

 ```javascript
 const crypto = require('crypto')
const http = require('http')
const mustache = require('mustache')
const getRawBody = require('raw-body')
const _ = require('lodash')
const flag = require('./flag')

const indexTemplate = `
<!doctype html>
<style>
  body {
    background: #172159;
  }
  * {
    color: #fff;
  }
</style>
<h1>your public blueprints!</h1>
<i>(in compliance with military-grade security, we only show the public ones. you must have the unique URL to access private blueprints.)</i>
<br>
{{#blueprints}}
  {{#public}}
    <div><br><a href="/blueprints/{{id}}">blueprint</a>: {{content}}<br></div>
  {{/public}}
{{/blueprints}}
<br><a href="/make">make your own blueprint!</a>
`

const blueprintTemplate = `
<!doctype html>
<style>
  body {
    background: #172159;
    color: #fff;
  }
</style>
<h1>blueprint!</h1>
{{content}}
`

const notFoundPage = `
<!doctype html>
<style>
  body {
    background: #172159;
    color: #fff;
  }
</style>
<h1>404</h1>
`

const makePage = `
<!doctype html>
<style>
  body {
    background: #172159;
    color: #fff;
  }
</style>
<div>content:</div>
<textarea id="content"></textarea>
<br>
<span>public:</span>
<input type="checkbox" id="public">
<br><br>
<button id="submit">create blueprint!</button>
<script>
  submit.addEventListener('click', () => {
    fetch('/make', {
      method: 'POST',
      headers: {
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        content: content.value,
        public: public.checked,
      })
    }).then(res => res.text()).then(id => location='/blueprints/' + id)
  })
</script>
`

// very janky, but it works
const parseUserId = (cookies) => {
  if (cookies === undefined) {
    return null
  }
  const userIdCookie = cookies.split('; ').find(cookie => cookie.startsWith('user_id='))
  if (userIdCookie === undefined) {
    return null
  }
  return decodeURIComponent(userIdCookie.replace('user_id=', ''))
}

const makeId = () => crypto.randomBytes(16).toString('hex')

// list of users and blueprints
const users = new Map()

http.createServer((req, res) => {
  let userId = parseUserId(req.headers.cookie)
  let user = users.get(userId)
  if (userId === null || user === undefined) {
    // create user if one doesnt exist
    userId = makeId()
    user = {
      blueprints: {
        [makeId()]: {
          content: flag,
        },
      },
    }
    users.set(userId, user)
  }

  // send back the user id
  res.writeHead(200, {
    'set-cookie': 'user_id=' + encodeURIComponent(userId) + '; Path=/',
  })

  if (req.url === '/' && req.method === 'GET') {
    // list all public blueprints
    res.end(mustache.render(indexTemplate, {
      blueprints: Object.entries(user.blueprints).map(([k, v]) => ({
        id: k,
        content: v.content,
        public: v.public,
      })),
    }))
  } else if (req.url.startsWith('/blueprints/') && req.method === 'GET') {
    // show an individual blueprint, including private ones
    const blueprintId = req.url.replace('/blueprints/', '')
    if (user.blueprints[blueprintId] === undefined) {
      res.end(notFoundPage)
      return
    }
    res.end(mustache.render(blueprintTemplate, {
      content: user.blueprints[blueprintId].content,
    }))
  } else if (req.url === '/make' && req.method === 'GET') {
    // show the static blueprint creation page
    res.end(makePage)
  } else if (req.url === '/make' && req.method === 'POST') {
    // API used by the creation page
    getRawBody(req, {
      limit: '1mb',
    }, (err, body) => {
      if (err) {
        throw err
      }
      let parsedBody
      try {
        // default values are easier to do than proper input validation
        parsedBody = _.defaultsDeep({
          publiс: false, // default private
          cоntent: '', // default no content
        }, JSON.parse(body))
      } catch (e) {
        res.end('bad json')
        return
      }

      // make the blueprint
      const blueprintId = makeId()
      user.blueprints[blueprintId] = {
        content: parsedBody.content,
        public: parsedBody.public,
      }

      res.end(blueprintId)
    })
  } else {
    res.end(notFoundPage)
  }
}).listen(80, () => {
  console.log('listening on port 80')
})

 ```

At user creation, the flag is stored in one of its blueprint.
Because the blueprint is private, we need its ID to see it and get the flag.

 We looked at vulnerabilities in the modules in package.json.
 We saw that loadash as a vulnerability on ```defaultsDeep``` method allowing to make prototype pollution: https://snyk.io/test/npm/lodash/4.17.11

This method is used to set default values on created blueprint: 
```javascript
  try {
        // default values are easier to do than proper input validation
        parsedBody = _.defaultsDeep({
          publiс: false, // default private
          cоntent: '', // default no content
        }, JSON.parse(body))
      } catch (e) {
        res.end('bad json')
        return
      }
```

So we can use the prototype pollution attack in order to make all blueprint public.

Sending the payload: ```{"constructor": {"prototype": {"public": true}}}``` via POST to  ```/make``` API gave us the flag


Flag: flag{8lu3pr1nTs_aRe_tHe_hiGh3s1_quA11tY_pr0t()s}