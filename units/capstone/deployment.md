# Deploying a Full-Stack App using Heroku

## Resources

*Note: We are not following any of these guides precisely. These are the resources we used and remixed to deploy in a way that makes sense to us.*

* [Example on Reed's GitHub](https://github.com/crymall/react-aws-image-example)
* [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)
* [Deploy React and Express app to Heroku](https://daveceddia.com/deploy-react-express-app-heroku/)
* [Deploy a Postgresql database to Heroku](https://medium.com/@HalahSalih/how-to-deploy-an-express-app-to-heroku-with-postgresql-database-using-git-266e100d59ff)
* [Stack Overflow question on seeding a Heroku database with a .sql file](https://stackoverflow.com/questions/15237366/how-to-execute-a-sql-script-on-heroku)

## Introduction

Welcome to the fun, awesome world of deployment. You made stuff that lives in the browser - now we're gonna make it live on the Internet!

A couple things to remember:

* Don't just copy + paste stuff. Try to develop a real understanding of what each step is doing.
* Your app is going to break a lot. Either you'll see Heroku-branded errors, or you'll see a blank screen. Don't get upset - expect it and handle it.

With that said, let's get started.

## Step 1: Make a Heroku account and install the Heroku CLI

Alright. First, let's go to [Heroku](https://www.heroku.com/) and sign up for an account. The signup process should be pretty self-explanatory.

Heroku uses Git to manage its projects, so we're going to do a fair amount of stuff in the command line. Therefore, we want to [install](https://devcenter.heroku.com/articles/heroku-cli) the Heroku CLI, which can be done as a download, with Brew, or even (scroll down on that page) with NPM, in a pinch.

Next, close your terminal, open a new terminal window, and type `which heroku`. You should see a directory. That's where your Heroku files are installed.

Then, go to your project's root folder in the terminal and throw out a quick:

```
heroku create
```

...And we're on our way.

## Step 2: Edit your Express app

We're going to assume here that you are using create-react-app and express-generator. This also assumes that you are creating your React app **inside your Express app (as a subfolder)**. Don't fret if you have "frontend" and "backend" folders separate - all you have to do is change the paths that we'll outline here, and it should work just fine.

Your first task is to edit a few basic elements of express-generator. Take a look at `app.js`. You know that big chunk of middleware? Where you set up logger and bodyParser? We are going to edit the pathing of that very slightly:

```js
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'client/build'))); //this is the only thing that's different - 'client/build'
```

This simply says 'okay, Express - don't render your Express views here, we want our React app to render instead.' Basically, we're making React and Express work together on a single port.

To that end, we'll make one other change to our `app.js` file. After our backend routing, we place this snippet of code:

``` js
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname + '/client/build/index.html'));
});
```

This just makes sure we're never showing Express responses without the window dressing of React. JSON is, perhaps, not the best look for a production app to have. Nor is an error view on Express.

We are pretty close! In Express, all that's left is to set up our `package.json` file. Heroku automatically runs certain scripts if we name them properly. Therefore, there's nothing to do here besides run the proper commands in a particularly named script:

```js
"scripts": {
    "start": "node ./bin/www",
    "heroku-postbuild": "cd client && npm install --only=dev && npm install && npm run build"
  }
```

...And that should be it! All the edits we need to make to our Express app are done.

Well, besides...

## Step 3: Set up your Postgres database on Heroku

In the terminal, in your project's root folder, type the following:

```
heroku addons:create heroku-postgresql:hobby-dev -a <app-name-on-heroku>
```

This does a couple of things. It:

* Creates a Postgres database in our Heroku account, and
* Sets an environment variable DATABASE_URL that we can use to access our database using pg-promise. Type `heroku config` to see your unique database URL.

Now, there's two ways to structure and seed our database. You can either access the command to open your database in the terminal on database.heroku.com, manually typing in `CREATE` and `INSERT` statements, or you can use a .sql file. We are going to be doing the latter.

First, we have to edit our .sql file very slightly. No longer are we `DROP DATABASE`-ing, nor do we have to `CREATE` one or `\c` into it, because we will already be accessing a database we created from Heroku. Everything afterwards (`DROP/CREATE TABLE`) we can keep.

After making those changes, we type the following into the command line in our `db` folder:

```
cat file.sql | heroku pg:psql
```

Where `file.sql` is our .sql file's name.

Barring any issues, your database should be seeded! There's only one last thing to do. We go to our `queries.js` file and instead of all that `localhost` stuff to reference our database, we write `process.env.DATABASE_URL`. Like so:

```js
var pgp = require('pg-promise')({});
var connectionString = process.env.DATABASE_URL;
var db = pgp(connectionString);
```

...And we are all set!

## Step 4: Deploy that sucker

Would you believe me if I said that the only thing you have to do now is:

```
git add .
git commit -m 'heroku version'
git push heroku master
```

...Because that's it.

Okay, maybe not *quite* it. If you want to keep your Heroku version in a different branch, now would be the time to make one, commit to it, and instead of `git push heroku master`, write:

```
git push heroku yourbranch:master
```

Barring any issues (and there can be issues!) sit tight for, like, 10 minutes, and after that, you should see your site on Heroku at the URL it gives you!

### Troubleshootin'

Okay, so you've deployed, you've waited ten minutes, but your app broke. Or maybe you assume it broke because you only see a blank screen. Not to worry! Heroku is very particular about syntax - much moreso than localhost - so you should expect a bump in the road here or there.

The main thing to do is to **figure out where the problem is.** Messages telling you about issues in your code can be found in three different places:

* The terminal itself when you `git push` to Heroku. This is pretty easy - it usually gives you an explicit error, a filename, and even a specific line.
* Chrome Developer Tools. This tends to have a lower success rate, but sometimes it can be useful. It's especially useful to know if anything was even sent to the frontend. Look through the HTML and JS for anything familiar.
* Your Heroku backend. Navigate to a project in Heroku, click the upper-right corner where it says "More," and click "View Logs." You should see a verbose description of what, exactly, is going on with your app at any given time.

Don't understand an issue, or can't see what's causing the problem? There are a million and one reasons why a Heroku app might not compile. If it's a weird error message, **Google it!** If it's a generic error message, but you still can't figure out what's wrong, get another set of eyes on your code. Don't bang your head against something for nothing!
