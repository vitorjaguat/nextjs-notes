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

##### Environment variables + MongoDB

In this section, Max hides the password by keeping the Git repository private. I wanted to make my repository public, so I added an environment variable `MY_ENVIRONMENT_VARIABLE` on the Vercel project page and set it to `<the MongoDB connect string>`. This value is a string, but you should NOT write it between '' in Vercel. Just the content itself.

Then I referred to process.env.`MY_ENVIRONMENT_VARIABLE` in the MongoClient.connect function. This way the string with the password wasn't exposed in the git repository.

To use the variable locally, I created a `.env` file and set `MY_ENVIRONMENT_VARIABLE='<the MongoDB connect string>'`. Restarted the dev server and it worked. Here you should use ''.

Also remember to add `.env` to `.gitignore`. Now we can push our code to GH as a public repo.

##### Environment variables + Firebase

1. Create `.env.local` file on your project and write:

```js
NEXT_PUBLIC_API_KEY = your_api_key;
NEXT_PUBLIC_AUTH_DOMAIN = your_auth_domain;
NEXT_PUBLIC_PROJECT_ID = your_project_id;
NEXT_PUBLIC_STORAGE_BUCKET = your_storage_bucket_url;
NEXT_PUBLIC_MESSAGING_SENDER_ID = your_messaging_id;
NEXT_PUBLIC_APP_ID = your_auth_id;
NEXT_PUBLIC_DATABASE_URL = your_db_url;
```

2. Update `firebase/config.js` file:

```js
const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_APP_ID,
  databaseURL: process.env.NEXT_PUBLIC_DATABASE_URL,
};
```

3. When deploy to Vercel, set the environment variables on the project's Settings:
   ![on the project's settings](/img/next-env.webp)

4. Make sure to uncheck the Automatic Expose checkbox.
   ![on the project's settings](/img/next-env2.webp)

5. Ready to deploy the app!

### MISSING POINTS:

- authenticating using Next.js / MongoAtlas

## Internationalization: multilingual webapps

The best package to serve SSR using Next.js is 'next-i18next'.

1. Install
   `npm i next-i18next react-i18next i18next`

2. Setup 1: create a `next-i18next.config.js` file in root:

```js
module.exports = {
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'pt'],
  },
};
```

3. Setup 2: add i18n to `next.config.js`:

```js
const { i18n } = require('./next-i18next.config');

module.exports = {
  i18n,
};
```

4. Create provider in `_app.js`:

```js
import { appWithTranslation } from 'next-i18next';

const MyApp = ({ Component, pageProps }) => <Component {...pageProps} />;

export default appWithTranslation(MyApp);
```

5. In all page files, add `serverSideTranslations(locale, ['filenames', 'filenames'])` to the props key returned by `getStaticProps`:

```js
import { serverSideTranslations } from 'next-i18next/serverSideTranslations';

export async function getStaticProps({ locale }) {
  return {
    props: {
      ...(await serverSideTranslations(locale, [
        'common',
        'footer', //these are the names of json files
      ])),
      otherPropKey: 'otherpropvalue',
      // Will be passed to the page component as props
    },
  };
}
```

It's also possible to do the same when using `getServerSideProps`.

6. Create files with translation content. By default, next-i18next expects your translations to be organised as such:

```js
.
└── public
    └── locales
        ├── en
        |   └── common.json
        └── de
            └── common.json
```

Inside the lang folders, there can be many files, don't forget to pass them all to `serverSideTranslations` and to call them when using the `useTranslation` hook (see next step).

Each json file must be structured like this:

```js
{
  "navbar": {
    "home": "home",
    "about": "about",
    "skills": "skills",
    "projects": "projects",
    "contact": "contact"
  },
  "home": {
    "title": "LET'S BUILD SOMETHING TOGETHER",
    "hi": "Hi, I'm ",
    "name": "Hugh",
    "subtitle": "A Front-End Web Developer",
    "text": "I'm a front-end web-developer specializing in building (and also designing) exceptional digital experiences. Currently, I'm focused on building responsive front-end web-applications while learning back-end technologies."
  }
}
```

7. Using the translated content with `useTranslation`:

```js
import { useTranslation } from 'next-i18next';

export const Home = () => {
  const { t } = useTranslation('common');

  return (
    <footer>
      <p>{t('about.description')}</p>
    </footer>
  );
};
```

We can use `useTranslation` in any component, but `serverSideTranslations` only have to be used in page components.

8. Changing languages. To toggle between languages, we can create a simple ToggleLanguage component. In that, we need to use `useRouter` to get the info about the correct route, and the `i18n` object returned by `useTranslation()` to access the `changeLanguage(locale)` method. The handler function must first change the language, then redirect the user to the correct path. The /lang (eg, /pt, /en) paths are automatically generated by Next.js as part of its SSR.

```js
import { useTranslation } from 'next-i18next';
import { useRouter } from 'next/router';

const LanguageToggle = () => {
  const { i18n } = useTranslation();
  const router = useRouter();

  const handleLanguageChange = (locale) => {
    i18n.changeLanguage(locale); //changing the language
    router.push(
      {
        pathname: router.pathname,
        query: router.query,
      },
      router.asPath,
      { locale, scroll: false }
    ); //redirect to the new language's path, eg, '/pt/#home'
  };

  //router.push(url, as, options)
  //url: the url to navigate to
  //as: optional decorator (#home, eg)
  //options: object of options:
  //scroll: Optional boolean, controls scrolling to the top of the page after navigation. Defaults to true
  //shallow: Update the path of the current page without rerunning getStaticProps, getServerSideProps or getInitialProps. Defaults to false
  //locale - Optional string, indicates locale of the new page

  return (
    <div>
      <button
        className='p-1 mr-2 mb-6 text-sm'
        onClick={() => handleLanguageChange('en')}
      >
        EN
      </button>
      <button
        className='p-1 text-sm'
        onClick={() => handleLanguageChange('pt')}
      >
        PT
      </button>
    </div>
  );
};

export default LanguageToggle;
```

## SMALL BITS: Next.js

### Fonts from Google Fonts

https://www.youtube.com/watch?v=L8_98i_bMMA&ab_channel=LeeRobinson
The most traditional way to import Google Fonts into our project is to copy the link tag from Google Fonts and add inside our Head component. But there is another way that is more efficient:

1. `npm i @next/font`
2. Import the desired font: `import { Nunito } from '@next/font/google';`
3. Instantiate the font and define subsets and weight. Then add the className into the tag where you want to apply the font (can be the main tag for the whole app also):

```js
//_app.js
import { Nunito } from '@next/font/google';

const nunito = Nunito({
  subsets: ['latin'],
  weight: ['400', '700'],
}); //instantiate the font

export default function App({ Component, pageProps }) {
  return (
    <main className={nunito.className}>
      {' '}
      //add the className
      <Navbar />
      <Component {...pageProps} />
    </main>
  );
}
```

### Local fonts

https://www.youtube.com/watch?v=L8_98i_bMMA&ab_channel=LeeRobinson
To add a locally stored font, it's basically the same as adding a font from Google Fonts.

1. `npm i @next/font`
2. Import localFont: `import localFont from '@next/font/local'`
3. Instantiate the font: `const myFont = localFont({ src: './myfont.woff2' });`
4. Then add the className into the tag where you want to apply the font (can be the main tag for the whole app also).

### Smooth scrolling

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

### Protecting routes

To add protected routes in Next.js, we can use the useEffect hook:

```js
//new-post.js
const { user } = useAuthContext();
const router = useRouter();

useEffect(() => {
  user ? router.push('/new-post') : router.push('/signup');
}, [user]);
```

Now, even if the user tries to insert the route /new-post in the browser, as soon as he reaches the page, before loading the jsx, useEffect will check if the the user is logged in.

# Tailwind CSS

## Setup

## Adding custom colors (variables)

_repo portfolio-nextjs_
We can add custom colors (primary, secondary) and define our themes, just like using CSS variables.

1. Go to `tailwind.config.js` and add new custom color names inside of the extend object. We can use any name that we wish.
2. Now we can use these color names in any class that accepts a color parameter. Eg: bg-primary text-secondary etc

```js
//tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: '#c1c1c1',
        secondary: '#909090',
        tertiary: '#242424',
      },
    },
  },
  plugins: [],
};

//


#### On Markdown

- **preview**: shift + command + v
- **same page links**: [here](#place-2)

_repo react-max-23-nextjs, lecture 318_
```
