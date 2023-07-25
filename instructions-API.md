## Create the back-end
1. Create a new space in the Storyblok app

## Create the front-end
1. Create a new project
```
npx nuxi@latest init <project-name>
cd <project-name>
npm install
npm run dev
```

## Fetch API

1. In `pages>index.vue`: initialize variable to store data; use a fetch request to get the data from storyblok.
```
<template>
    <h1>index.vue</h1>
</template>

<script>
  export default {
      data: function(){
          return{
              story:''
          }
      },
      created: function() {
          fetch('https://api.storyblok.com/...')
          .then(resp => resp.json())
          .then(data => this.story = data.story.content)
      }
  }
</script>
```
2. Delete `app.vue`

## Render data
1. Call the component
```
<Teaser :body="story.body" />
```

2. Create component components>Teaser.vue
```
<template>
    <h2>Teaser component</h2>
    <div v-if="body" class="card">
        <h2>{{ body[0].headline }}</h2>
    </div>

</template>

<script>
    export default {
        props:['body']
    }
</script>

<style>
    .card{
        margin-left: 2rem;
    }
</style>
```

## Multiple elements
1. Loop the body and use the conditional render to display the components
```
<template v-for="block in story.body" :key="block._uid">
        <Card v-if="block.component === 'card'" :blok="block" />
        <Teaser v-if="block.component === 'teaser'" :blok="block" />
</template>
```


## Fetch Wordpress
1. In `pages>index.vue`, change the access URL and render a list of posts
```
fetch('https://wptavern.com/wp-json/wp/v2/posts?_fields=author,id,excerpt,title,link')
```
```
<ol>
    <div v-for="post in posts" :key="post.id">
        <li>
            <NuxtLink :to="`/post/${post.id}`" ><h2 v-html="post.title.rendered"></h2></NuxtLink >
            <p v-html="post.excerpt.rendered"></p>
        </li>
    </div>
</ol>
```
2. Create the blog component pages/post/[id].vue
```
<template>
    <div class="post-container">
        <h1 v-if="post.title">{{ post.title.rendered }}</h1>
        <div v-if="post.content" v-html="post.content.rendered"></div>
    </div>
</template>

<script>

export default{
    data: function(){
        return{
            id: this.$route.params.id,
            post:''
        }
    },
    created: function(){
        fetch(`https://wptavern.com/wp-json/wp/v2/posts/${this.id}?_fields=author,id,content,title,link`)
          .then(resp => resp.json())
          .then(data => this.post = data )
    }
}

</script>

<style>
img{
    max-width: 100%;
}
.post-container{
    max-width: 800px;
    margin: 0 auto;
}
</style>
```

## Storyblok API
*delete app.vue
1. Install the official SDK for Nuxt 3
```
npm install @storyblok/nuxt
```
2. In nuxt.config.js, set the basic configuration : accessToken and region  

```
modules: [['@storyblok/nuxt', { accessToken: '' }]],
apiOptions: {
    region: "us",
},
```
3. Load content using the Storyblok API
- pages/index.vue

```
<script setup>
    const story = await useAsyncStoryblok('home', { version: 'draft' })
</script>
```

```
<template>
    <StoryblokComponent v-if="story" :blok="story.content" />
</template>
```

- ./storyblok/Page.vue
```
<template>
    <h1>Here the Page.vue</h1>
      <div>
        <StoryblokComponent v-for="blok in blok.body" :key="blok._uid" :blok="blok" />
      </div>
</template>

<script>
    export default {
        props:['blok']
    }
</script>
```

- ./storyblok/Card.vue
```
<template>
    <h3>Card - storyblok API</h3>
    <div>
      Card title: {{ blok.title }}
    </div>
</template>
   
<script>
    export default {
        props:['blok']
    }
</script>
```
- ./storyblok/Teaser.vue
```
<template>
    <h3>Teaser - storyblok API</h3>
    <div>
      Teaser headline: {{ blok.headline }}
    </div>
</template>
   
<script>
    export default {
        props:['blok']
    }
</script>
```

