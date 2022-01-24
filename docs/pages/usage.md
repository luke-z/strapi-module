---
title: Usage
description: 'Learn how to use the Strapi module in your Nuxt app'
position: 3
category: Guide
---

## Authentication

To handle authentication in your Nuxt app with Strapi, you can:

### Login

```js
await this.$strapi.login({ identifier: '', password: '' })
```

### Register

Custom user fields, if added to the entity, can be sent with the registration payload as well. For example `phoneNumber`.

```js
await this.$strapi.register({ email: '', username: '', password: '', phoneNumber: '' })
```

### Logout

```js
await this.$strapi.logout()
```

### Forgot password

```js
await this.$strapi.forgotPassword({ email: '' })
```

### Reset password

```js
await this.$strapi.resetPassword({ code: this.$route.query.code, password: '', passwordConfirmation: '' })
```

### User

Once logged in, you can access your `user` everywhere:

```vue{}[components/navbar.vue]
<template>
  <div>
    <p v-if="$strapi.user">
      Logged in
    </p>
  </div>
</template>

<script>
export default {
  middleware: 'auth',
  computed: {
    user () {
      return this.$strapi.user
    }
  },
  methods: {
    logout () {
      this.$strapi.logout()
      this.$router.push('/')
    }
  }
}
</script>
```

You can for example use a [Nuxt middleware](https://nuxtjs.org/docs/2.x/directory-structure/middleware) to protect pages when the user is not logged in:

```js{}[middleware/auth.js]
export default function ({ redirect, $strapi }) {
  if (!$strapi.user) {
    return redirect('/login')
  }
}
```

> See all the [available methods](/strapi#methods)

## Entities

You can access the default Strapi CRUD operations by using these methods:

- `find`
- `count`
- `findOne`
- `create`
- `update`
- `delete`

```js
await this.$strapi.find('products')
```

> See more in [$strapi methods](/strapi#methods)

You can also define your Strapi entities to add shortcuts to the previous methods, see [options](/options#entities).

```js
await this.$strapi.$products.find()
```
## Usage with the Nuxt Composition Api

First you need to install the [Nuxt Composition API](https://composition-api.nuxtjs.org/) and add it to your project

In order to use the Composition API you must be on Nuxt v2.12 or above because we will be using the useFetch Hook instead of useAsync due to the fact that useAsync [won't re call the information after inital load](https://composition-api.nuxtjs.org/helpers/useAsync)

Your setup method will look like this

```
import { useFetch, ref } from '@nuxtjs/composition-api'
export default{
  setup() {
          const homepage = ref()
          useFetch(async ({ $strapi }) => {
              homepage.value = await $strapi.find('home')
          })
          return { homepage }
      },
}
```
> :warning: Remember that the JSON data is case sensitive

If you want to use dynamic route matching you can pass in the url parameters by doing this

```
import { ref, useFetch, useRoute } from '@nuxtjs/composition-api'
export default {
    name: '_id',
    setup() {
        const article = ref()
        const route = useRoute()
        useFetch(async ({ $strapi }) => {
            article.value = await $strapi.find('articles', {
                slug: route.value.params.id,
            })
        })

        return { article }
    },
}
```

## Advanced

### Accessing $http

If you defined custom routes in your Strapi API that goes out of the REST scope, this module exposes `$http`:

```js
this.results = await this.$strapi.$http.$get('/products/search', { searchParams: { _q: 't-shirt' } })
```

### Updating current user

<alert type="info">

You often need to update your user, and so on define a custom route in Strapi: `PUT /users/me`.

</alert>

You can use this module to call it this way:

```js
const user = await this.$strapi.$users.update('me', form)
```

And then mutate the `user`:

```js
this.$strapi.user = user
```
