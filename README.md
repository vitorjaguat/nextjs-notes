# Next.js

What is Next.js?

- "The React framework for production"
- A **fullstack** framework for React: that means, it builds up on React, and it makes building large-scale React apps easier.

### Next.js' key-features:

1. Server-side rendering

In React apps, typically, the page is loaded on the fly: when a user clicks to open a post, for example, the data is loaded and the page is "mounted" on the fly. The Search Engine Crawlers are unable to be aware of our page's content.

Server-side rendering allows us to pre-render React pages and components on a server. thus solving the above mentioned problem. Another advantage of server-side rendering is that it avoids flickering and "loading..." states as we first load a page.

More about the difference between plain React and Next.js regarding SSR: https://www.imaginarycloud.com/blog/next-js-vs-react/

2. File-based Routing

With React Router, we define routes by using code. With Next.js, we define pages and routes with files and folders instead of code.

This implies less code, less work and highly understandable and readable structure.

3. Fullstack capabilities

- easily add backend (server-side) code to your apps
- storing data, getting data, authentication etc. can be added to your React projects

### Creating a Next.js app

`npx create-next-app`

This will give us a structure containing the folders node_modules, styles, public, pages, etc. The most important folder in a Next.js app is `pages`.

The pages folder contains an index.js file. This file exports a React component that will be rendered whenever the user reaches `our-domain.com/` path.

A file named `news.js` in the pages folder will be rendered whenever the user reaches `our-domain.com/news` path.

A folder created inside of the pages folder also will have its index.js rendered whenever the user reaches `our-domain.com/foldername` path. This approach is important if we want to create nested routes, like `our-domain.com/news/sport` for example.

This is how the routing works in a Next.js app.

### Dynamic pages (with parameters)

To create a dynamic page, that act like a template to render different content, we will name that page file using square brackets: `pages/news/[newsId].js`. This file will be rendered whenever the user reaches a path like `our-domain.com/news/something`.

To get the value of 'something', we will use a custom hook, which is built-in inside of `next/router`: `useRouter`.

```js
//[newsId].js
//ourdomain.com/news/something will render this page.

import { useRouter } from 'next/router';

function DetailPage() {
  const router = useRouter();

  const newsId = router.query.newsId;

  //send a request to the backend API
  // to fetch the news data with that Id

  return <h1>The Detail Page</h1>;
}

export default DetailPage;
```

### Linking between pages

In order to link between pages (inside a navbar for example), we can use the Link component imported from `next/link`.

While React's Link component takes a `to=` prop, the Next.js Link component takes a `href=` prop.

```js
<Link href='/news/bolsonaristas-cagam-na-cadeira-do-xandao'>
  Bolsonaristas cagam na cadeira do Xandão
</Link>
```

Navbar:
_repo react-max-23-nexjs2_

```js
//components/MainNavigation.js
import classes from './MainNavigation.module.css';
import Link from 'next/link';

function MainNavigation() {
  return (
    <header className={classes.header}>
      <div className={classes.logo}>React Meetups</div>
      <nav>
        <ul>
          <li>
            <Link href='/'>All Meetups</Link>
          </li>
          <li>
            <Link href='/new-meetup'>Add New Meetup</Link>
          </li>
        </ul>
      </nav>
    </header>
  );
}

export default MainNavigation;
```

### The \_app.js file

In Next.js apps, the `_app.js` file is a special file. It must be placed in the pages folder of our app. It serves like a root component, that will wrap every single page inside our app.

Inside \_app.js, `Component` and `pageProps` are standard objects, that will contain the components rendered inside of MyApp component, and the props of that component.

```js
import Layout from '../components/layout/Layout';
import '../styles/globals.css';

function MyApp({ Component, pageProps }) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  );
}

export default MyApp;
```

### Programatic navigation (redirect)

The next/router hook `useRouter` has a method `push('/path')` that allows us to navigate programatically to that path, ie, redirect the user to that path.

```js
import { useRouter } from 'next/router';

etc etc
const router = useRouter();
router.push('/');
etc etc
```

### getStaticProps(): serving pre-rendered pages

Whenever we are creating a static website, the source code will appear without the data fetched from an API, if we use plain React. This is bad for search-engine-crawlers.

However, in Next.js, we have two forms of pre-rendering:

1. Static Site Generation (SSG): good for static websites, or dynamic websites whose database won't change multiple times per second => here, the webpages are pre-rendered on the build process
2. Server-side Rendering: good for webapps whose database change everytime (dynamic websites) => here, the pages are rendered by the srever every time there is a request for that page

In a static website, we can configure the app to serve pages that are pre-rendered via static generation. We do that by exporting the async function `getStaticProps()` in our pages.

The code that will run in that function (usually fetching data and handling this data) will only run during the `building process` of our pages, ie, this code won't be stored in our client-side app.

`getStaticProps()` must return an object that contains a props object. That props object will be the props of that particular page.

Alternatively, a plain React app would use useEffect without dependencies to fetch the data from a server. But this would result in an 'empty' source code, not good for search-engine purposes.

```js
export default function HomePage(props) {
  return <MeetupList meetups={props.meetups} />;
}

export async function getStaticProps() {
  //this code will only run on the build process
  //this code will never appear on the client-side

  //fetch data from an API

  return {
    props: {
      meetups: DUMMY_MEETUPS, //these properties are received by the page component as its own props
    },
    revalidate: 10, //this SSG-page will be re-generated every 10s on the server (if the page receives requests)
  };
}
```

**revalidate** property: without this property, the page is built as a SSG-page on the build process, and stays the same after that. If, for example, the API receives new POST requests, the website will remain outdated. The **revalidate** property, inside the object returned by `getStaticProps()`, solves this problem. The number set as its value is the period of time (in seconds) in which our page will be re-generated (inside the server), at least when this page receives new requests.

#### getStaticPaths(): working with params for SSG data fetching

When working with params (in dynamic pages like \[itemId\].js), we can extract the requested param using the `context.params` property. In order to do that, we have to pass the context object to the getStaticProps() function:

```js
export async function getStaticProps(context) {
  //fetch data for a single meetup

  const { meetupId } = context.params; //requested param!

  return {
    props: {
      meetupData: {
        image: , //this data can be added after we fetch data and filter the response using params
        id: ,
        title: ,
        address: ,
        description:
      }
    }
  }
}
```

When working with params, besides `getStaticProps()` we also must export another function in our page component: `getStaticPaths()`.

getStaticPaths() must return an object with two properties:

- fallback: if false, the paths object contains all possible params and ids; if true or 'blocking', it can generate missing ones on the fly.
- paths: \[array\] contains one property:
  - params: {object} contains one property:
    - paramName: eg, meetupId, contains one property:
      - pathName: eg, 'm1', the path that identifies the data to be fetched.

(it's easier to see the example:)

```js
export async function getStaticPaths() {
  return {
    fallback: 'blocking', //if false, the paths object contains all possible params and ids; if true or 'blocking', it can generate missing ones on the fly.
    paths: [
      {
        params: {
          meetupId: 'm1',
        },
      },
      {
        params: {
          meetupId: 'm2',
        }, //later on, we're going to fetch all meetupIds from the server, so that we don't need to hard code every single one.
      },
    ],
  };
}
```

When reaching dymanic path that isn't mapped in getStaticPaths:
fallback: false => will show an error page
fallback: true => will show an empty page, generate a complete page on the server-side, then show the complete page
fallback: 'blocking' => won't show anything until server-side has generated a complete page, then it'll show it (generally the best choice)

### getServerSideProps() and Server-side rendering (SSR)

If we can't get satisfied updating our pre-rendered page only when building or every X seconds, we can then export `getServerSideProps()`. This function will only run on the server every time a request for that page is made.

The disadvantage of getServerSideProps() is that it can take longer than getStaticProps(), as we have to wait for the server to generate the page everytime we load it.

getServerSideProps() also offers access to the `req` and `res` objects, which come as properties of the `context` parameter. This can be useful for authentication middlewares, to handle session data, etc.

```js
export async function getServerSideProps(context) {
  const req = context.req;
  const res = context.res;

  //fetch data from an API, etc.

  return {
    props: {
      meetups: DUMMY_MEETUPS,
    },
  };
}
```

### API Routes

return to [here](#getstaticpaths-working-with-params-for-ssg-data-fetching) to complete and add link there

Next.js offers the possibility to add code to create a complete API, inside of our project, thus allowing us to build a fullstack application.

#### Creating a MongoAtlas db and fetching a POST request

1. Create an `api` folder inside of `pages` folder. All the code written here will not be served in the client-side, only in the server-side, thus it's possible to add credentials like api-keys here.
2. Create a .js file with a name that describe what will happen there. Eg, 'new-meetup.js'.
3. On the browser, set up a MongoDB Atlas cluster and click 'connect'.
4. npm install the official MongoDB driver via Terminal: `npm i mongodb`.
5. Write the code in our /api/new-meetup.js file:

```js
// pages/api/new-meetup.js
// when a request is sent to /api/new-meetup, Next.js will trigger this function:

import { MongoClient } from 'mongodb';

async function handler(req, res) {
  if (req.method === 'POST') {
    const data = req.body; //getting the req.body, which will contain the form data (title, image, address, description)

    //handle errors would be good!

    const client = await MongoClient.connect(
      'mongodb+srv://<username>:<password>@react-nextjs.2wgcrxv.mongodb.net/meetups-db?retryWrites=true&w=majority'
    ); //add username and password where needed; the name 'meetups-db' after 'mongodb.net/' is the name of our db, it'll be created on the fly as we use it.
    const db = client.db('meetups-db');

    const meetupsCollection = db.collection('meetups'); //this is the name if your collection inside 'meetups-db' database. it will be created on the fly if doesn't exist.

    const result = await meetupsCollection.insertOne(data); //inserting new document into the collection.

    console.log(result);

    client.close(); //closing the connection;

    res.status(201).json({ message: 'Meetup inserted!' }); // giving a response with a custom message.
  }
}

export default handler;
```

6. Write the code where you want to send the POST request:

```js
// pages/new-meetup/index.js
// our-domain.com/new-meetup/

import NewMeetupForm from '../../components/meetups/NewMeetupForm';

export default function NewMeetupPage() {
  async function addMeetupHandler(enteredMeetupData) {
    const response = await fetch('/api/new-meetup', {
      method: 'POST',
      body: JSON.stringify(enteredMeetupData),
      headers: {
        'Content-Type': 'application/json',
      },
    });

    const data = await response.json();

    console.log(data);
  }

  return <NewMeetupForm onAddMeetup={addMeetupHandler} />;
}
```

#### Fetching a GET request

**FIRST CASE: getting all documents from a collection**

1. For GET request, we usually don't need to create a new file on the `api` folder. That's because, if we are using functions that only run server-side (like `getStaticProps()`) in our page component, we can insert the backend code directly into that server-side rendering function. This code, as we said, will not be exposed client-side.

```js
// src/index.js
// will be shown when hitting 'our-domain.com/'
import { MongoClient } from 'mongodb';
import MeetupList from '../components/meetups/MeetupList';

export default function HomePage(props) {
  return <MeetupList meetups={props.meetups} />;
}

export async function getStaticProps() {
  //this code will never appear on the client-side

  //fetch data from API:
  //fetch('/api/meetups') //we could do it like this, but we don't need to create a new file!
  const client = await MongoClient.connect(
    'mongodb+srv://jaguat:2rfcofge@react-nextjs.2wgcrxv.mongodb.net/meetups-db?retryWrites=true&w=majority'
  );
  const db = client.db('meetups-db');
  const meetupsCollection = db.collection('meetups');

  const meetups = await meetupsCollection.find().toArray(); //get all documents in 'meetups' collection and transform them into an array;

  client.close();

  return {
    props: {
      meetups: meetups.map((meetup) => ({
        title: meetup.title,
        address: meetup.address,
        image: meetup.image,
        id: meetup._id.toString(), //convert the ObjectId() generated by MongoDB to a string;
      })),
    },
    revalidate: 10,
  };
}
```

**SECOND CASE: getting one document and preparing a dynamic-path rendering**

1. First, we need to generate our array of paths in `getStaticPaths()` on the \[meetupId\].js page dynamically. We can do it by getting an array of all \_id's from our documents in our MongoDB collection. Then, we map this array to create the array of paths that must be returned:

```js
// pages/[meetupId]/index.js
etc etc
export async function getStaticPaths() {
  const client = await MongoClient.connect(
    'mongodb+srv://jaguat:2rfcofge@react-nextjs.2wgcrxv.mongodb.net/meetups-db?retryWrites=true&w=majority'
  );
  const db = client.db('meetups-db');
  const meetupsCollection = db.collection('meetups');

  const meetups = await meetupsCollection.find({}, { _id: 1 }).toArray(); //get only the _id of each document and put them into an array.

  client.close()

  return {
    fallback: false, //if false, the paths object contains all possible params and ids; if true, it can generate missing ones on the fly.
    paths: meetups.map((meetup) => ({
      params: { meetupId: meetup._id.toString() },
    })) //generate our array of paths dynamically.
  };
}
etc etc
```

2. Fetch one single document from the database using getStaticProps() and pass it through props to the component.

```js
// pages/[meetupId]/index.js
import { MongoClient, ObjectId } from 'mongodb';
import MeetupDetail from '../../components/meetups/MeetupDetail';

export default function MeetupDetails(props) {
  return (
    <MeetupDetail
      image={props.meetupData.image}
      title={props.meetupData.title}
      address={props.meetupData.address}
      description={props.meetupData.description}
      id={props.meetupData.id}
    />
  );
}

export async function getStaticPaths() {
  see above
}

export async function getStaticProps(context) {
  // getting the param from the context object:
  const { meetupId } = context.params; //

  //fetch data for a single meetup:
  const client = await MongoClient.connect(
    'mongodb+srv://jaguat:2rfcofge@react-nextjs.2wgcrxv.mongodb.net/meetups-db?retryWrites=true&w=majority'
  );
  const db = client.db('meetups-db');
  const meetupsCollection = db.collection('meetups');

  const selectedMeetup = await meetupsCollection.findOne({
    _id: ObjectId(meetupId),
  }); // don't forget to import { ObjectId } from 'mongodb'!

  client.close();

  return {
    props: {
      meetupData: {
        id: selectedMeetup._id.toString(), //converting ObjectId to a string.
        title: selectedMeetup.title,
        address: selectedMeetup.address,
        description: selectedMeetup.description,
        image: selectedMeetup.image,
      },
    },
  };
}
```

### Adding head with title and description

HTML head tag contains metadata about our website, like title and description. This metadata will provide search engines with information about our website.

We can add this information using the Head component provided by 'next/head'.

```js
// /pages/index.js
import Head from 'next/head';
import MeetupList from '../components/meetups/MeetupList';

export default function HomePage(props) {
  return (
    <>
      <Head>
        <title>Meetups</title>
        <meta
          name="description"
          content="Your website to create meetups all around the world"
        />
      </Head>
      <MeetupList meetups={props.meetups} />
    </>
  );
}

etc etc
```

We can add the Head component in all page components. Alternatively, we can add a single Head component inside the \_app.js root component. Then, all pages will have the same metadata:

```js
// pages/_app.js
import Head from 'next/head';
import Layout from '../components/layout/Layout';
import '../styles/globals.css';

function MyApp({ Component, pageProps }) {
  return (
    <>
      <Head>
        <title>Meetups</title>
        <meta
          name='description'
          content='Your website to create meetups all around the world'
        />
      </Head>
      <Layout>
        <Component {...pageProps} />
      </Layout>
    </>
  );
}

export default MyApp;
```

We can also add dynamic values inside the Head component's tags. This is especially useful in dynamic pages (templates that use params), for example, pages that contain details about items on a list:

```js
import Head from 'next/head';
import MeetupDetail from '../../components/meetups/MeetupDetail';

export default function MeetupDetails(props) {
  return (
    <>
      <Head>
        <title>{props.meetupData.title}</title>
      </Head>
      <MeetupDetail
        image={props.meetupData.image}
        title={props.meetupData.title}
        address={props.meetupData.address}
        description={props.meetupData.description}
        id={props.meetupData.id}
      />
    </>
  );
}

etc etc
```

### Deploying Next.js projects

Next.js was created by the Vercel team, then this hosting service is optimized for hosting Next.js projects. We just have to import our GitHub repo to Vercel, and it will run build and build our compiled files for us.

As our project contains credentials, it is better to make the GitHub repo private, then link it to Vercel. About environmental variables, see below.

#### Configuring MongoDB Network Access

You will have to `allow access from anywhere` in MongoDB Network Access settings. This is important so that Vercel can access MongoDB to get new data and pre-render pages, store new data from forms, etc.

#### Updating dependencies

Sometimes our project was written with dependencies that are not uptodate to current Vercel standards, so maybe we should update them in our project before deployment, by running `npm install next@12 react@18 react-dom@18 --force`.

Sometimes you also have to follow these steps:

First remove all node moudle folder and packge-lock.json, then change the `package.json` (see below) and `npm i` again, then finally Vercel deploy will work:

```js
 {
  "name": "nextjs-course",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "mongodb": "^3.6.4",
    "next": "12.1.4",
    "react": "17.0.2",
    "react-dom": "17.0.2"
  }
}
```

#### Environment variables

In this section, Max hides the password by keeping the Git repository private. I wanted to make my repository public, so I added an environment variable `MY_ENVIRONMENT_VARIABLE` on the Vercel project page and set it to `<the MongoDB connect string>`. This value is a string, but you should NOT write it between '' in Vercel. Just the content itself.

Then I referred to process.env.`MY_ENVIRONMENT_VARIABLE` in the MongoClient.connect function. This way the string with the password wasn't exposed in the git repository.

To use the variable locally, I created a `.env` file and set `MY_ENVIRONMENT_VARIABLE='<the MongoDB connect string>'`. Restarted the dev server and it worked. Here you should use ''.

Also remember to add `.env` to `.gitignore`. Now we can push our code to GH as a public repo.

### MISSING POINTS:

- authenticating using Next.js / MongoAtlas

### SMALL BITS: Next.js

#### Smooth scrolling

```js
// globals.css
html {
  scroll-behavior: smooth;
}

// on any file that contains Link:
<Link href='#something' scroll={false}>Something</Link>

//on the components being linked (add id):
export default function Projects() {
  return (
    <div id='projects' className='w-full'>
    etc etc
  )}

```

#### On Markdown

- **preview**: shift + command + v
- **same page links**: [here](#place-2)

_repo react-max-23-nextjs, lecture 318_
