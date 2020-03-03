<p align="center">
    <a href="https://github.com/chimera-js/vue-chimera">
        <img src="https://github.com/chimera-js/vue-chimera/raw/master/VueChimeraLogo.png" width="300px">
    </a>
</p>
<h1 align="center">Vue Chimera</h1>

[![vuejs](https://img.shields.io/badge/vue.js-2.x-green.svg)](https://vuejs.org)
[![circle ci](https://img.shields.io/circleci/project/github/chimera-js/vue-chimera/master.svg)](https://circleci.com/gh/chimera-js/vue-chimera)
[![npm version](https://img.shields.io/npm/v/vue-chimera.svg)](https://www.npmjs.org/package/vue-chimera)
[![npm downloads](https://img.shields.io/npm/dt/vue-chimera.svg)](http://npm-stat.com/charts.html?package=vue-chimera)
[![npm bundle size (minified + gzip)](https://img.shields.io/bundlephobia/minzip/vue-chimera.svg)](https://bundlephobia.com/result?p=vue-chimera@1.0.0)
[![npm bundle size (minified + gzip)](https://img.shields.io/bundlephobia/minzip/vue-chimera.svg)](https://bundlephobia.com/result?p=vue-chimera@1.0.0)

VueJS RESTful client with reactive features.
Vue-Chimera is based on [axios](https://github.com/axios/axios) http client library.

Overview of features: 
- Reactive endpoints and requests based on vue instance data
- Loading flags
- Get time when the resource was latestly loaded
- Set interval time to refresh data on an endpoint
- Simple robust resource definition
- Nuxtjs compatible with Server side prefetching
- Simple cancelation on requests
- Cancel all pending requests on vue instance destroy
- Events and handlings
- Lightweight

## Demo
[Demo: vue-chimera](https://cdn.rawgit.com/chimera-js/vue-chimera/f4312e49/example/simple/index.html)

## Installing

Using npm:

```bash
$ npm install vue-chimera
```

Using bower:
```bash
$ bower install vue-chimera
```

Using cdn:
```html
<script src="https://unpkg.com/vue-chimera@latest/dist/vue-chimera.js"></script>
```

## Getting started

To add **vue-chimera** to your Vue you must use it as a plugin:
*ECMAScript 6*
```javascript
import Vue from 'vue'
import VueChimera from 'vue-chimera'

Vue.use(VueChimera)

```

If you use old browser style just add `vue.min.js` and `dist/vue-chimera.js` script tag to your HTML file and everything would be fine to go on.

#### Defining Resources
To use Vue Chimera in your vue instance you should add `chimera` property to root of your vue component instance:

```javascript
let app = new Vue({
  
  // Chimera property contains all of chimera restful queries/resources and options
  chimera: {
    
    // Here you can define your restful resources and api endpoints
    mySampleResource: '/users'
  },
  
  data() {
    return { ... }
  }
  
})
```

Vue Chimera automatically converts your `resources` to [Resource](https://github.com/chimera-js/vue-chimera/blob/master/src/Resource.js) Object
Your resources can be:
* A simple **string** for simple GET requests
* An **Object** for complex resources like: POST, PATCH, with Parameters, with Headers, Response/Error transformer, Event listeners
* An instance of **Resource**
* A **Function** for reactive resources [Reactive-Resources](#reactive-resources)

```javascript

var app = new Vue({
    
    chimera: {
      
      // Define options and configs inside `$options` special key
      $options: {
        // You can use axios config to set default config for your axios http client library
        axios: {
          baseURL: 'https://my-domain.com/api/v1',
          headers: {
            'X-Sample-Access-Token': 'xxx'
          }
        },
          
        // Or you can directly pass a axios client for more control over your client
        axios: axios.create(),
      },
      
      // Define resources inside chimera
      users: '/users',
        
      time: {'url': '/time',
        // With interval option set to 5000, resource will be refreshed every 5000 miliseconds
        'interval': 5000
      },
        
      // a sample POST request
      sendPost: {
        url: '/posts',
        method: 'POST',
        params: {
          title: 'Sample',
          body: '<h1>Vue Chimera is awesome...</h1>'
        },
          
        // Set prefetch to false to prevent request from sending on application load.
        prefetch: false
      }
      
    },
    
    data() {
      return {
        selectedUserID: 1
      }
    }
    
})

```

Now it's time to use our resources in our vue template or inside vue methods.

#### Use Resources in template

`chimera` instance can be accessed in templates with `$chimera` (Dollar sign prepended) or you can get resources by simply type it's name, It had been injected in computed properties by the plugin.
`data` is the final json result of our restful resource
`loading` is a boolean flag, identifies the resource is loading.
You can read other resource property and methods [here](#resource-properties-and-methods).

```html
<template>
  <div>
    <!-- `users` is Resource object defined in chimera (same as: `$chimera.users`) -->
    <ul v-if="users.data && !users.loading">
      <li v-for="user in users.data">
        {{ user.name }}
      </li>
    </ul>
    <small v-else>Loading...</small>
  </div>
</template>
```

#### Use Resources in script

You can simply access `chimera` instance with `$chimera`

```javascript
let app = new Vue({
  
  methods: {
    
    sendUser() {
      
      // Execute function on resources, sends the request and returns a Promise
      this.$chimera.sendUser.execute().then(res => {
          
          // You can do other things after request success here.
          
      }).catch(err => {
          // Or error handlings
      })
    
    }
    
  }

})
```

#### Resource properties and methods

| Property | Type  | Default Value | Description
| -------- | :-----: | :-------------: | -----------
| data     | Object/ Array |  null | The resource response object or string returned from server when request is successfull
| loading  | Boolean | false       | Indentifies the resource is in loading state 
| error    | Object/string  | null | Error json object or string returned from server when request failed
| lastLoaded | Date         | null | The date/time from last time resource successfully loaded (null if not loaded yet)
| status    | number        | null | Resource response status
| headers    | Object        | null | Resource response/error headers

|   Method   | Return type | Description
| ---------- | ----------- | -----------
| reload(force)    | Promise   | Fetches the resource from server. `force`: True for cache busting
| execute(force)  | Promise   | Same as Reload
| send(extraParams)  | Promise   | Sends request with extra data
| on(event, handler)|           | Sets an event listener. [Events](#events)
| cancel()         | void       | Interupts request
| startInterval()  | void       | Manually starts interval (auto refresh)
| stopInterval()         | void       | Manually stops interval


#### Reactive Resources
You can also set a **function** to a resource that will return **String**, **Object**, **instance of Resource** same as before,
to let your resources be reactive and change.
```javascript
let app = new Vue({

    data() {
      return {
        selectedUserId: 1,
        postId: 2
      }
    },

    chimera: {
        
      post() {
        return {
          url: '/api/v1/posts',
          params: {
            postId: this.postId
          },
          method: 'post',
          prefetch: true
        }
      },
        
      user: () => `/api/v1/users/${this.selectedUserId}`
    }

})
```

_Note that `prefetch` evaluates to false, your data will be reactive and change but won't be fetched until you call the `reload` function of the resource_

#### Chimera instance properties
```javascript
...
  methods: {
    send() {
      
      // Global identifier to check if any request is pending
      this.$chimera.$loading
      
      // Get the axios client to modify or ...
      this.$chimera.$axios
      
      // Cancels all requests (Automatically called on instance destory)
      this.$chimera.$cancelAll()
    }
  }
...
```

#### Transformers
Transformers is used to change the response to another format. It would be called before the request response (error or success) is mapped to the `data` attribute of Resource object.

```javascript
new Vue({
  
  chimera: {
    users: {
        
      url: '/users',
    
      transformer: {
        response: (response) => {
          if (response.user)
            response.user.id = 'UID: ' + response.user.id
          return response
        },
        error: (error) => {
          error.msg = error.msg || 'Something went wrong'
          return error
        }
      }
    }
  }
  
})
```

#### Events

```javascript
// Events: cancel, error, success, loading, timeout

new Vue({

    chimera: {
      users: {
        url: '/users',
        on: {
          'cancel': (resource) => {
              // Calls when a request interrupted and cancelled
          }
        }
      }
    },
    
    methods: {
      // Or listen programmatically
      listenToError() {
        this.$chimera.users.on(EVENT_ERROR, function(err) {
          alert('Oops error occured! status: ' + err.status)
        })
      }
    }
    
})

```


## Using with Nuxt.js
You can use Vue-Chimera with nuxtjs to use it's SSR features so you can easily prefetch the data.
```javascript
// nuxt.config.js

module.exports = {
  
  modules: [
    'vue-chimera/nuxt'
  ],
  
  chimera: {
    // Server side prefetch will only be available for resources that has `prefetch` and `ssrPrefetch`
    prefetch: 'get',
    
    // Enables server side prefetch on resources
    // true: fetched on server
    // false: fetched on client
    // 'override': fetched on server and client (overrided by client)
    ssrPrefetch: true,
    
    ssrPrefetchTimeout: 2000 // Server side timeout for prefetch
  }
  
}
```

You can also disable SSR for some heavy resources
```javascript
...
    chimera: {
      myResource: {
        url: '/api/v1/example',
        ssrPrefetch: false // Prefetch disabled on a specific resource
      }
    }
...
```

## Maintainer
<p>
<a href="https://github.com/SasanFarrokh" target="_blank" rel="noopener noreferrer"><img src="https://avatars1.githubusercontent.com/u/20913428?s=460&v=4" width="200"></a>
</p>

## Contribution
This project is open for any contribution. We are waiting for your pull requests.
Thanks to all our contributors.

## Roadmap
* Vue Directives
* Different Cache implementations
* Vuex Implementation
* Vuejs web components

## License
[MIT](https://github.com/chimera-js/vue-chimera/blob/master/LICENSE.MD)
