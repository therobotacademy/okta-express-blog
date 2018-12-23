# [Step-by-step tutorial](https://developer.okta.com/blog/2018/06/28/tutorial-build-a-basic-crud-app-with-node)
https://developer.okta.com/blog/2018/06/28/tutorial-build-a-basic-crud-app-with-node

# Index

  - [Create Your Express.js App](#create-your-expressjs-app)
  - [Initialize Authentication](#initialize-authentication)
  - [Install Dependencies](#install-dependencies)
  - [Define Database Models with Sequelize](#define-database-models-with-sequelize)
  - [Initialize Your Express.js App](#initialize-your-expressjs-app)
      - [Initialize Node.js Middlewares](#initialize-nodejs-middlewares)
      - [Initialize Node.js Routes](#initialize-nodejs-routes)
      - [Initialize Error Handlers](#initialize-error-handlers)
  - [Create Express.js Views](#Create-Express.js-Views)
  - [Create Styles](#Create-Styles)
  - [Create Routes](#Create-Routes)
    - [Create User Routes](#Create-User-Routes)
    - [Create Blog Routes](#Create-Blog-Routes)
       - [Create an Authentication Helper](#Create-an-Authentication-Helper)
       - [Create the Homepage](#Create-the-Homepage)
       - [Create the Dashboard Routes](#Create-the-Dashboard-Routes)
       - [Create the Edit Routes](#Create-the-Edit-Routes)
       - [Create the Delete Route](#Create-the-Delete-Route)
       - [Create the Display Route](#Create-the-Display-Route)
   - [Test Your New CRUD App!](#Test-Your-New-CRUD-App!)


# Create Your Express.js App
[Back to index](#Index)


`$ sudo npm install -g express-generator`

```
$ express --view pug blog
$ npm install
```
Run the app:
```
$ DEBUG=blog:* npm start
```
Go to your browser and access the recently launched Express website:

`http://localhost:3000`

# Initialize Authentication
[Back to index](#Index)

- Once you’ve logged into your **Okta Developer Portal** and land on the dashboard page, copy down the Org URL located at top right of the screen.
- Then create a new application by browsing to the Applications tab and clicking Add Application.
- Select Web and on the settings page, enter the following values:
   - Name: Blog
   - Base URIs: http://localhost:3000
   - Login redirect URIs: http://localhost:3000/users/callback
   - You can leave all the other values unchanged

Now that your application has been created, copy down the Client ID and Client secret values on the following page, you’ll need them soon.

- Finally, create a new authentication token. This will allow your app to talk to Okta to retrieve user information, among other things.
- To do this, click the API tab at the top of the page followed by the Create Token button.
- Give your token a name, preferably the same name as your application, then click Create Token. Copy down this token value as you will need it soon.

# Install Dependencies
[Back to index](#Index)

The first thing you need to do in order to initialize your Express.js app is install all of the required dependencies.
```
npm install express@4.16.3
npm install @okta/oidc-middleware@0.1.2
npm install @okta/okta-sdk-nodejs@1.1.0
npm install sqlite3@4.0.1
npm install sequelize@4.38.0
npm install async@2.6.1
npm install slugify@1.3.0
npm install express-session@1.15.6
```


# Define Database Models with Sequelize
[Back to index](#Index)

The first thing I like to do when starting a new project is define what data my application needs to store, so I can model out exactly what data I’m handling.

Create a new file named `./models.js` and copy it from the parent Github repo.

This code initializes a new SQLite database that will be used to store the blog data and also defines a model called Post which stores blog posts in the database. Each post has a title, a body, an author ID, and a slug field.
The call to `db.sync();` at the bottom of the file will automatically create the database and all of the necessary tables once this JavaScript code runs.

# Initialize Your Express.js App
[Back to index](#Index)

The next thing I like to do after defining my database models is to initialize my application code. This typically involves:
- Configuring application settings
- Installing middlewares that provide functionality to the application
- Handling errors

Open the file `./app.js` and replace its content with that of the parent Github repo.
Be sure to replace the placeholder variables with your actual Okta information.
- Replace `{yourOktaOrgUrl}` with the Org URL on your dashboard page
- Replace `{yourOktaClientId}` with the Client ID on your application page
- Replace `{yourOktaClientSecret}` with the Client secret on your application page
- Replace `{aLongRandomString}` with a long random string (just mash your fingers on the keyboard for a second)

You can do this externally using environment variables to keep unchanged the `app.js` file:
```
# These variables are to be used with file 'app.js'
export OKTA_ORG_URL=https://dev-113242.oktapreview.com
export OKTA_CLIENT_ID=0oaijcb5z2uGlpptn0h7
export OKTA_CLIENT_SECRET=cJebv0gI3xnQTJAx17tH7IZciP14qWt-4D4aXfJb
export OKTA_SECRET=y51fXUC1OakD4d
export OKTA_TOKEN=00uOOZFjFChLd5YmOGkN05tYIAEtCS7oKXl6-iKSuq
```
Check values:
```
$ printenv | grep OKTA
```
Yo may integrate the load of environment variables into the Node app by using the [dotenv module](https://github.com/motdotla/dotenv):
```
$ npm install dotenv
```
Place `require('dotenv').config();` in the header of your `app.js` file, and put the enviroment variables into a file named `.env`.  Add environment-specific variables on new lines in the form of NAME=VALUE. For example:
```
DB_HOST=localhost
DB_USER=root
DB_PASS=s1mpl3
```

You can use the `--require (-r)` command line option to preload dotenv. By doing this, you do not need to require and load dotenv in your application code. This is the preferred approach when using import instead of require:
```
$ node -r dotenv/config your_script.js
```

## Initialize Node.js Middlewares
[Back to index](#Index)

Middlewares in Express.js are functions that run on every request. There are many open source middlewares you can install and use to add functionality to your Express.js applications. The code in `app.js` uses several popular Express.js middlewares, as well as defines some new ones.

1. The first few middlewares are all standard stuff: they enable logging, parse form data, and serve static files. The interesting thing to take note of is the use of the `ExpressOIDC` middleware.
 - `app.use(logger("dev"));`
 - `app.use(express.json());`
 - `app.use(express.urlencoded({ extended: false }));`
 - `app.use(express.static(path.join(__dirname, "public")));`
 - `const oidc = new ExpressOIDC({`

2. This middleware handles the OpenID Connect authentication logic of the application which supports login, logout, etc. The settings being passed into the `ExpressOIDC` middleware are configuration options which dictate what URLs are used to log the user into the application, and where the user will be redirected once they’ve been logged in.
 - `const oidc = new ExpressOIDC({`

4. The next middleware is the `session` middleware. This middleware is responsible for managing user cookies and remembering who a user is. The `secret` it takes must be a long random string you define and keep private. This secret makes it impossible for attackers to tamper with cookies.
 - `app.use(session({`

5. The `oidc.router` middleware uses the settings you defined when creating `ExpressOIDC` to create routes for handling user authentication. Whenever a user visits `/users/login`, for instance, they’ll be taken to a login page. This line of code is what makes that possible.
 - `app.use(oidc.router);`

6. Finally, there’s a custom middleware. This middleware creates a `req.user` object that you will be able to use later on to more easily access a currently logged in user’s personal information.
 - `app.use((req, res, next) => {`

## Initialize Node.js Routes
[Back to index](#Index)

Also in the code in `app.js`.

The route code tells Express.js what code to run when a user visits a particular URL.
This code tells Express.js that in our (yet to be created) blog and user route files there are functions that should be executed when certain URLs are hit. If a user visits a URL starting with `/users`, Express.js will look for other matching URLs in the user routes file. If a user visits any URLs starting with the / URL, Express.js will look in the blog routes file to see what to do.

## Initialize Error Handlers
[Back to index](#Index)

The last bit of code in our app above is the error-handling middlewares.
These middlewares will run if any 4XX or 5XX type errors occur. In both cases, they will render a simple web page to the user showing them the error.

# Create Express.js Views
[Back to index](#Index)

Views in Express.js are the equivalent of HTML templates—they’re the place you store front-end code and logic. The views you’ll use in this project will use the `Pug` templating language which is one of the most popular.
Remove your existing views by running the following command:
```
$ rm views/*
```

- Create a `./views/layout.pug` file. This is a base “layout” template that all other templates will inherit from. It defines common HTML, includes the Bootstrap CSS library, and also defines a simple navigation menu.
- Create the `./views/error.pug` file. This page will be shown when an error occurs.
- Create the `./views/unauthenticated.pug` file. This page will be shown when a user tries to visit a page but they aren’t logged in.
- Now define the `./views/index.pug` template. This is the homepage of the website and lists all the current blog posts ordered by date.
- The next view to define is `./views/post.pug` which displays a single blog post.
- Now create the file `./views/edit.pug` which contains the blog post editing page markup.
- Finally, create `./views/dashboard.pug` which will render the dashboard page that users will see once they’ve logged in. This page allows a user to create a new post as well as edit and delete their existing posts.
 
# Create Styles
[Back to index](#Index)

Since CSS is straightforward and not the focus of this tutorial, you can simply copy the CSS below into the `./public/stylesheets/style.css` file.

# Create Routes
[Back to index](#Index)

Routes are where the real action happens in any Express.js application. They dictate what happens when a user visits a particular URL.

To get started, remove the existing routes that the express-generator application created.
```
$  rm routes/*
```

Next, create the two route files that you’ll need.
```
$  touch routes/{blog.js,users.js}
```
- The `./routes/blog.js` file will contain all of the routes related to blog functionality.
- The `./routes/users.js` file will contain the routes related to user functionality.

While you could always put all your logic in the main `./app.js` file, keeping your routes in separate purpose-based files is a good idea.

## Create User Routes
[Back to index](#Index)

Since **Okta’s oidc-middleware library** is already handling user authentication for the application, there isn’t a lot of user-facing functionality we need to create.

The only route you need to define that relates to user management is a logout route — this route will log the user out of their account and redirect them to the homepage of the site. While the oidc-middleware library provides a logout helper, it doesn’t create an actual route.

Copy and place the file `./routes/users.js`

The way to understand this route is simple. When a user visits the `/logout` URL, a function will run that uses the **oidc-middleware library** to log the user out of their account Redirects the now logged-out user to the homepage of the site.

## Create Blog Routes
[Back to index](#Index)

Since the application you’re building is a blog, the last big piece of functionality you need add is the actual blog route code. This is what will dictate how the blog actually works: how to create posts, edit posts, delete posts, etc.

Copy the `./routes/blog.js` file.

*NOTE: Make sure you substitute in your values for the placeholder variables towards the top of this file. You need to replace {OKTA_ORG_URL} and {OKTA-TOKEN} with the appropriate values.*

Let’s take a look at each route and how it works:

### Create an Authentication Helper
[Back to index](#Index)

The first function you’ll notice in the blog routes is the `ensureAuthenticated` function.

This function is a special middleware you’ll use later on that will render the `unauthenticated.pug` view you created earlier to tell the user they don’t have access to view the page unless they log in.

This middleware works by looking for the `req.user` variable which, if it doesn’t exist, means that the user is not currently logged in. This will be helpful later on to make sure that only logged in users can access certain pages of the site (for instance, the page that allows a user to create a new blog post).

### Create the Homepage
[Back to index](#Index)

The `index` route `(aka: “homepage route”)` is what will run when the user visits the root of the site. It will display all blog posts ordered by date and not much else.

The way this works is by first using `Sequelize.js` to retrieve a list of all blog posts from the database ordered by the `createdAt` field. Whenever a new blog post is stored in the database, `Sequelize.js` automatically assigns it both a createdAt and updatedAt time field.

Once a list of posts have been returned from the database you will iterate over each post retrieving it in JSON format, then use Okta’s Node SDK to retrieve the author’s information via the authorId field.

Finally, you’ll build an array consisting of all the blog posts alongside the author’s name, and will render the index.pug template which then takes that data and displays the full web page.

### Create the Dashboard Routes
[Back to index](#Index)

The dashboard page is the first page users will see after logging in. It will:
 - Allow users to create a new blog post
 - Show users a list of their previously created blog posts
 - Provide buttons that allow a user to edit or delete previously created blog posts

Note that there are technically two routes here:
- The first route function is run when a user issues a `GET` request for the `/dashboard` page, while
- the second route is run when a user issues a `POST` request for the `/dashboard` page.

The first route retrieves a list of all blog posts this user has created, then renders the dashboard page:
- Note how it uses the `ensureAuthenticated` middleware we created earlier (pladec inside `helpers.js` file).
- By inserting the `ensureAuthenticated` middleware into the route, this guarantees that this route code will only execute if a currently logged in user is visiting this page.

If a user chooses to create a new blog post, that will trigger a `POST` request to the `/dashboard` URL, which is what will eventually run the second dashboard route shown above.
- This route uses `Sequelize.js` to create a new database entry storing the blog posts and author details, then renders the dashboard page once more.

### Create the Edit Routes
[Back to index](#Index)

The edit routes control the pages that allow a user to edit one of their existing blog posts.

These routes by matching a variable pattern URL:
- If the user visits a URL that looks like `/<something>/edit`, then the edit route will run.
- Because the URL pattern in the route is defined as `/:slug/edit`, Express.js will pass along the URL route in the  `req.params.slug` variable so you can use it.

These routes handle rendering the edit page as well as updating existing posts when needed.


### Create the Delete Route
[Back to index](#Index)

*NOTE: This route is not including `Helpers.ensureAuthenticated` !!!!!!!!!!*

The delete route is simple: if a user sends a `POST` request to the URL `/<post-url>/delete`, then `Sequelize.js` will destroy the post from the database.

### Create the Display Route
[Back to index](#Index)

The display route is the simplest of them all. It renders a specific blog post on a page. It works much like the other routes above by using variable URL patterns.

When a user visits a URL like `/my-great-article`, this route will run, query the database for any blog posts whose slug is `my-great-article`, then display that post on a page.

# Test Your New CRUD App!
[Back to index](#Index)

By this point, you’ve built a fully functional Node.js website using Express.js and Okta. To test it out, run the following command to start up your web server and then visit in the browser:
```
$ npm start

http://localhost:3000
```