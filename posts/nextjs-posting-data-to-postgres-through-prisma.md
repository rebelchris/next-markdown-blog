---
title: 'Next.js posting data to Postgres through Prisma'
metaTitle: 'Next.js posting data to Postgres through Prisma'
metaDesc: 'Posting data to Postgres using Prisma in Next.js'
socialImage: images/27-10-2021.jpg
date: '2021-10-27'
tags:
  - nextjs
  - prisma
---

Hi everyone! In the past couple of articles, we have been looking at Prisma and Postgres databases.

In this article, I will create a Next.js app that can post data to a Postgres database.

What we'll be building:

- User signs in with Spotify
- User loads their playlists from Spotify
- User can sync one of the playlists to our Postgres database

It will look like this:

<!-- ![Next.js posting data to Postgres through Prisma](https://cdn.hashnode.com/res/hashnode/image/upload/v1634477800714/AYIILKoPa.gif) -->
<video autoplay loop muted playsinline>
  <source src="https://res.cloudinary.com/daily-dev-tips/video/upload/q_auto/prisma-post_ocdqqw.webm" type="video/webm" />
  <source src="https://res.cloudinary.com/daily-dev-tips/video/upload/q_auto/prisma-post_galhvt.mp4" type="video/mp4" />
</video>

## Setting up the starting point

I'm going to use the [Spotify login example](https://daily-dev-tips.com/posts/retrieving-a-persons-spotify-playlist-in-nextjs/) we made yesterday as the starting point for today's article.

If you want to follow along, download it from [GitHub](https://github.com/rebelchris/next-spotify-login) here.

The first thing we need to do is add the Prisma dependencies to our application.

```bash
npm i -D prisma
npm i @prisma/client
```

Then we need to initialize the Prisma client.

```bash
npx prisma init
```

This will generate the Prisma folder and add a database URL to our `.env` file.

Open up the `.env` file and paste your [Postgres database URL](https://daily-dev-tips.com/posts/setting-up-a-free-postgresql-database-on-heroku/).

## Set up the database

The next thing we need to do is define a schema for our playlist. Open the `prisma/schema.prisma` file and add the following schema at the bottom.

```js
model Playlist {
  id           Int @default(autoincrement()) @id
  title        String
  image        String?
  uri          String @unique
  addedBy      String
}
```

From here, we need to build our database.

```bash
npx prisma db push
```

As well as generate the local schema:

```bash
npx prisma generate
```

## Creating an API endpoint to post our entity

We already have a `playlists` endpoint so let's leverage that one but modify it to accept `POST` requests.

Open the `pages/api/playlists.js` file and start by importing the Prisma requirements.

```js
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();
```

Now let's modify the handler to do something on `POST` and `GET`.

```js
const handler = async (req, res) => {
  const {
    token: { accessToken, email },
  } = await getSession({ req });
  if (req.method === 'POST') {
    // Do post stuff
  } else if (req.method === 'GET') {
    const response = await getUsersPlaylists(accessToken);
    const { items } = await response.json();
    return res.status(200).json({ items });
  }
  res.end();
};
```

As for the `POST` section, we want to extract the correct data from our post query and create a new object to send to our database.

```js
if (req.method === 'POST') {
  const { body } = req;
  const {
    name,
    images: { 0: { url } = {} },
    uri,
  } = JSON.parse(body);
  const playlistItem = {
    title: name,
    image: url,
    uri: uri,
    addedBy: email,
  };
}
```

Then all we need to do is call our Prisma client and use the `create` function to insert our item.

```js
const playlist = await prisma.playlist.create({
  data: playlistItem,
});
return res.status(200).json(playlist);
```

And that's it, if we now perform a `POST` request to this API endpoint, our playlist will be added.

## Creating the frontend action

For the frontend part, let's open up our `index.js` page.
Inside the map function add a button with a click action like so:

```js
{
  list.map((item) => (
    <div key={item.id}>
      <h1>{item.name}</h1>
      <img src={item.images[0]?.url} width='100' />
      <br />
      <button onClick={() => saveToDatabase(item)}>Save in database</button>
    </div>
  ));
}
```

Now let's go ahead and make this `saveToDatabase` function.

```js
const saveToDatabase = async (item) => {
  const res = await fetch('api/playlists', {
    method: 'POST',
    body: JSON.stringify(item),
  });
  const data = await res.json();
};
```

In our case, we are just passing the API request but not doing anything with the return data yet.

This is perfect as once we click the button, it will call this function and post it to our API.
Which in return adds a new entry in our database.

![Database entries from Prisma Next.js](https://cdn.hashnode.com/res/hashnode/image/upload/v1634477581356/AS1_2nxU7.png)

You can also find the complete code on [GitHub](https://github.com/rebelchris/next-spotify-login/tree/post-data).

### Thank you for reading, and let's connect!

Thank you for reading my blog. Feel free to subscribe to my email newsletter and connect on [Facebook](https://www.facebook.com/DailyDevTipsBlog) or [Twitter](https://twitter.com/DailyDevTips1)
