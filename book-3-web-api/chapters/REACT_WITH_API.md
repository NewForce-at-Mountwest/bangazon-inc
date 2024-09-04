# Adding a React Client

> **NOTE:** Even though there isn't _technically_ a lot of new material in the next couple of chapters, at first all this is going to seem pretty foreign. You may find yourself looking over it and saying "Hmmm.... This looks vaguely familiar. Did we cover this? Did I used to know this? Did I dream it? Did I see it in a vision? Are these memories even mine?". You may find yourself scouring the dusty attic of your mind, looking under sheets, opening boxes...pausing now and again to reflect on your past and the array of choices that have led you to this time and place...trying to pinpoint exactly where things went wrong.  
> 
> Don't worry. This is all perfectly natural.  
> 
> You're gonna be fine.

---

## Objectives

After completing this lesson and working on related exercises you should be able to:

1. Define the term "client" as it relates to Web APIs.
1. List at least three types of applications that could be a client for a Web API.
1. Use configuration in a react app's `package.json` file to avoid CORS errors.
1. Write code in a react application to get a collection of data from an ASP<span>.</span>NET Core Web API.
1. Write code in a react application to add a new record using an ASP<span>.</span>NET Core Web API.

---

The cool thing about exposing a Web API is that any client capable of making HTTP requests can communicate with it. You can create web apps in React, Angular, or Vue and they could all talk to the same API. It even extends past browser based clients. You could make a mobile or desktop app and they could talk to the API. Maybe even IOT devices ([Internet of Things](https://en.wikipedia.org/wiki/Internet_of_things)) like microwaves or refrigerators want to make requests to your API.

## Create React App
 
Programming refrigerators may come in a later chapter. For now, lets build a react client that will make requests to our API. Open your Gifter project and make a directory called `client`. This is where we'll put our React code. In your terminal, `cd` into the client directory and run `npm create vite@latest . -- --template react`, `npm i react-router-dom` and then run `npm install`. Test to make sure react is working with `npm run dev` and click the counter.


**NOTE** VS Code has a much better developer experience when writing javascript. For our projects, it's recommended that you continue writing your C# code in Visual Studio, but use VS Code for javascript. `cd` into the newly created `client` folder and open VS Code from there using the `code .` command. These will affectionately be known as the blue one and the purple one. 

## Handling CORS

In development, our React app will be running on port 5173 and our API will be running on port 5001. Making requests from React to the Web API will result in CORS issues. We've seen one way we can resolve this--by enabling CORS on the server. Another way we can handle this is by adding a `proxy` property in our package.json file. Update the top of your package.json file to look like this:

```js
{
  "name": "client",
  "proxy": "https://localhost:5001",
  "version": "0.1.0",
  // ...other code omitted for brevity...
```

## Adding Post Provider

Make two directories under the `src` folder: `components` and `services`. In the services directory create a file called `PostService.jsx` and give it the following code

```js
import React from "react";

const baseUrl = '/api/post';

export const getAllPosts = () => {
  return fetch(baseUrl) 
    .then((res) => res.json())
};

export const addPost = (singlePost) => { 
  return fetch(baseUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(singlePost),
  });
};
```


This providers the state value of the posts array, as well as methods to fetch all posts and add a new post. Note that the urls we are making requests to are relative urls--they don't have anything like `https://localhost:5001/api/posts`. This is a benefit of adding the `proxy` attribute in our package.json file.

## Adding a Posts List Component

Inside the components directory, create a file called `PostList.jsx` and add the following code

```js
import React, { useState, useEffect } from "react";
import { getAllPosts } from "../services/PostService";

const PostList = () => {
  const [posts, setPosts] = useState([]);

  const getPosts = () => {
    getAllPosts().then(allPosts => setPosts(allPosts)); 
  };

  useEffect(() => {
    getPosts();
  }, []); 



  return (  
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <img src={post.imageUrl} alt={post.title} />
          <p>
            <strong>{post.title}</strong>
          </p>
          <p>{post.caption}</p>
        </div>
      ))}
    </div>
  );
};

export default PostList;
```

Nothing too fancy here. When the component loads, it will call the `getAllPosts` method it recieves from the provider and render a list of posts.

## Wiring It Up

We have our nice provider and component so lets use them. Replace main.jsx with the following code

```js
import React from "react";
import "./index.css";
import PostList from "./components/PostList";
import { BrowserRouter } from 'react-router-dom'

function App() {
 return (
<>
<BrowserRouter>
    <PostList />
  </BrowserRouter>
</>
)
}

export default App;
```
Now please go to Visual Studio Community (the purple one) and go into your solution explorer. Open up the properties folder and click into `launchSettings.json`. Replace the https code with this: 

```
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:5001;http://localhost:5013",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
```
Please relaunch your API (swagger) and run `npm run dev` in your terminal.

## Styling w/ Reactstrap

Lets make our app look a little nicer. Install reactstrap by running the following command (Make sure to run this from the same directory your package.json file is in)

```sh
npm install --save bootstrap reactstrap
```

Now import the css file into your `main.jsx` file

```js
import 'bootstrap/dist/css/bootstrap.min.css';
```

#### Making a Post Component

Our posts might get a little more complex so it's probably a good idea to separate it out into it's own component. Create a `Post.jsx` file in the components directory with the following code

```js
import React from "react";
import { Card, CardImg, CardBody } from "reactstrap";
import { Link } from "react-router-dom";


export const Post = ({ post }) => {
  return (
    <Card className="m-4">
      <p className="text-left px-2">Posted by: {post.userProfile.name}</p>
      <CardImg top src={post.imageUrl} alt={post.title} />
      <CardBody>
        <p>
          <Link to={`/posts/${post.id}`}>
          <strong>{post.title}</strong>
          </Link>
        </p>
        <p>{post.caption}</p>
      </CardBody>
    </Card>
  );
};
```

Again, nothing fancy here; we're just using the Card component that comes with reactstrap to organize some of the post details.

Now lets update the `PostList` component to use the new `Post` component

```js
import React, { useState, useEffect } from "react";
import { getAllPosts } from "../services/PostService";
import { Post } from "./Post";

const PostList = () => {
  const [posts, setPosts] = useState([]);

  const getPosts = () => {
    getAllPosts().then(allPosts => setPosts(allPosts)); 
  };

  useEffect(() => {
    getPosts();
  }, []); 
  return (
    <div className="container">
      <div className="row justify-content-center">
        <div className="cards-column">
          {posts.map((post) => (
            <Post key={post.id} post={post} />
          ))}
        </div>
      </div>
    </div>
  );
};

export default PostList;
```

## Exercise

1. Allow the user to add a new post. Create a `PostForm` component in the components directory and include it in main.jsx so that it shows up above the list of posts. Ex:

```js
import React from "react";
import "./index.css";
import PostList from "./components/PostList";
import { BrowserRouter } from 'react-router-dom'

function App() {
 return (
<>
<BrowserRouter>
    <PostList />
{/* Super Cool Component you just made here */}
  </BrowserRouter>
</>
)
}

export default App;
``` 

2. Add a "Search Posts" feature to your app that uses the `/api/post/search` API endpoint.

3. Update the Post component so that it includes the comments on each post.

    
    
