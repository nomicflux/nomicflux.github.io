---
title: Building a Blog with Servant and Opaleye, Part 1 - Setting Up the API
---

In the next series of blog posts, I'll be working through setting up a simple blog backend in Haskell using Servant for the server and Opaleye to manage database queries.  Aside from working through those two libraries, this is intended to be a beginner-intermediate tutorial, aimed at someone who has become familiar with the idea of monads and monad transformers (for example) but who would like a little bit of help in applying them to a real-world project.

For further information, you may want to check out the [Servant Tutorial](http://haskell-servant.readthedocs.io/en/stable/tutorial/index.html) and the [Opaleye Basic Tutorial](https://github.com/tomjaguarpaw/haskell-opaleye/blob/master/Doc/Tutorial/TutorialBasic.lhs).  You do not necessarily need to fully understand those tutorials in order to read this one; I wrote the current one myself to figure out what was what.

Note: This tutorial has been tested with Servant 0.8.  I'll try to make sure to come back every once in a while and update it to the current version as necessary.

## Step 0: Set It Up

Here, we'll set up a very basic Servant API.  It won't do much; our __GET__ requests will return hardcoded data, and our __POST__ requests will just append the posted data to the hardcoded response.  But we will be able to use almost all of the API functionality as we go forward.

If you want to create the project on your own, you can type `stack new blog-tutorial servant` at the command line.

Code for lesson 1 can be found at: [https://github.com/nomicflux/servant-opaleye-blog/tree/lesson1_servant_api](https://github.com/nomicflux/servant-opaleye-blog/tree/lesson1_servant_api).  (Note: if you do clone the code, make sure to `git checkout lesson1_servant_api` to get to the correct branch.)


## Step 1: Create App.hs

This is the file where I put basic information for the entire application.  For example, I get tired of typing `ExceptT ServantErr IO` all of the time, so I've created a type alias `AppM` instead.  Once we get to Lesson 4, we'll see that this setup will greatly ease our transition into more complex transformers.

Similarly, I've created typealiases for `BlogPostId` and `Email` as well.  In theory, the rest of the code should just have to know that it is dealing with *emails* and *ids*, and not worry about the underlying representation.  It is, of course, more complicated than that, since we'll also have to connect up Haskell's representation with the database and with JSON inputs, but this is a start toward full modularity.

## Step 2: Create API directory

The default setup provided by Stack places all of the API information in the Lib.hs file.  That's nice for a quick and dirty website, but we want to get maximum reuse out of our components.  We might be writing other websites which deal with users, for example - it happens from time to time.  So let's create a directory just for API files, and we'll get to work writing *API/User.hs* and *API/BlogPost.hs*.

## Step 3: Set up User API

### Datatype

To start, take the *User* information which Stack graciously provided in Lib.hs, and move it to it's own file.  We'll rename some things:  `API` becomes `UserAPI`, `server` becomes `userServer`.

To keep things simple, we will have just two fields for our *User*: an email (which will double as a unique identifier) and a password.  We've already set up the *Email* type alias in the App.hs file; we'll want to refer to *Email*s from the *BlogPost* API file as well, but we don't necessarily want *BlogPost*s to be dependent on our implementation of *User* beyond this one detail.  We end up with this:

```haskell
data User = User
    { userEmail    :: Email
    , userPassword :: String
    } deriving (Eq, Show)
```

### JSON

Next, let's create JSON representations.  We will use a different representation when converting to JSON as opposed to converting from JSON, so we'll have to roll our own `toJSON` and `parseJSON` functions.  When converting `toJSON`, we'll just package up the `userEmail` field - we should never return user passwords!
```haskell
instance ToJSON User where
    toJSON user = object [ "email" .= userEmail user ]
```

However, when using `parseJSON`, we'll take in both a `userEmail` and a `userPassword`:
```haskell
instance FromJSON User where
    parseJSON (Object o) = User <$>
                              o .: "email" <*>
                              o .: "password"
    parseJSON _ = mzero
```

If you have not used AESON before to convert to/from JSON, what we are doing is this: in `toJSON`, we set up an `object`, which matches up JSON keys with whatever we want.  Here, I lined up the key "email" with `userEmail user`, but I could have just set all emails to "bob@juno.com" if I felt like it.
```haskell
instance ToJSON BobUser where
    toJSON bob = object
        [ "email" .= "bob@juno.com"
        , "pasword" .= "Don't you want to know"
        , "extrafield" .= "I'm not even supposed to be here - " ++ userEmail user ]
```

To convert from JSON, we set up a `parseJSON` function, which takes an `object` and parses out the fields using `.:`.  So, for example, `object .: "email"` is something like `javascript_object.email` in javascript, which can then be used as part of a *User* datatype.  The main gotcha to watch out for is that AESON uses `Text` instead of `String`, so we have to add `{-# LANGUAGE OverloadedStrings #-}` if we don't feel like manually packing each `String`.

### API

What good is an API without the API itself?  We'll set up the endpoints as such:
```haskell
type UserAPI = Get '[JSON] [User]
          :<|> Capture "email" Email :> Get '[JSON] (Maybe User)
          :<|> ReqBody '[JSON] User :> Post '[JSON] [User]
```
We'll have a root endpoint which responds to a __GET__ and returns a List of *User*s in JSON format.  Next, we'll add an endpoint at "/{someone's email}" which will look up our current users and `Maybe` return one.  Finally, we'll let people __POST__ a *User* to our mini-server, and which will return a list of *User*s, also in JSON.

Then, we'll make sure to add a `Proxy` for our API:
```haskell
userAPI :: Proxy UserAPI
userAPI = Proxy
```
This is a little bit of boilerplate which lets the Type system interact with values which we pass around.  Basically, we can't send `UserAPI`, the Type, as an argument to anything, since it's not a value.  So instead, we send a `Proxy` in its place: `userAPI`.  Don't worry about it too much; by the time you'll need to do anything like this in your code (*if* ever), everything will be clear and you'll be writing the tutorials.

### Server

Above, we've set out our API.  We have our endpoints, what they expect from us, and what we expect from them.  This is really just setting up type signatures, however.  We'll need to implement a server to make those endpoints *do* something:
```haskell
userServer :: Server UserAPI
userServer = getUsers
        :<|> getUserByEmail
        :<|> postUser
```
Make sure that the server is in the same order as the API, otherwise the compiler will yell at you.

And if you give an API a Server, it will want some functions.  So let's set up some basic functions for the server as well:
```haskell
getUsers :: AppM [User]
getUsers = return users

getUserByEmail :: Email -> AppM (Maybe User)
getUserByEmail email = return $ listToMaybe $ filter ((== email) . userEmail) users

postUser :: User -> AppM [User]
postUser user = return $ users ++ [user]
```
As you'll notice, we are using `AppM` in our return value.  This was defined in App.hs as `ExceptT ServantErr IO`.  If you wanted to, you could type that in directly, and end up with type signatures such as `User -> ExceptT ServantErr IO [User]`.  But, a) that is a pain to read and type, and b) we'll be changing it in a future lesson, so abstracting the type out to `AppM` now will save time.

## Step 4: Set up BlogPost API

Repeat the above steps to now set up a blog post.  Create the file "API/BlogPost.hs".  For a datatype, we'll use:
```haskell
data BlogPost = BlogPost
              { bpId         :: BlogPostID
              , bpTitle      :: String
              , bpBody       :: String
              , bpUsersEmail :: Email
              , bpTimestamp  :: DateTime
              }
```
All of the steps will be the same as for the User API.  For an exercise, see how much you can implement on your own without looking at the BlogPost.hs file in the "src/API/" directory.

## Step 5: Put the APIs together

Now we have a User API and a BlogPost API, with their respective servers. We'll need to put them together for our main application.  Let's go back to Lib.hs, and add to the import list:
```haskell
import Api.User
import Api.BlogPost
```

Forming an API out of sub-APIs is no different than what we've already done:
```haskell
type API = "users" :> UserAPI
      :<|> "posts" :> BlogPostAPI

api :: Proxy API
api = Proxy

server :: Server API
server = userServer
    :<|> blogPostServer
```
We create the type by gluing together the sub-APIs and their respective endpoints.  Then we make a proxy and set up a server.  The server just refers back to the `userServer` and the `blogPostServer` we've already created.

Finally, we need to create an application to do the actual serving.  This should already be part of the Lib.hs code:
```haskell
app :: Application
app = serve api server

startApp :: IO ()
startApp = run 8080 app
```

## Step 6: Edit Cabal File

Ok, so the code is written, time to fire this thing up, right?  Not so fast - you could end up with some nasty, uninformative error messages at this point.

First, make sure that you add the modules we've been working on to the Cabal file.  These will be *App* (for the App.hs file holding cross-module information), *Api.User*, and *Api.BlogPost*, in addition to *Lib* which Stack has started you off with.  Put these in the *exposed-modules* section.

Second, add in all the depencies we've used.  If you forget one, Cabal will let you know.  Check the blog-tutorial.cabal file for the specific ones I've added.

## Step 7: Run

At this point, you should have a working Servant server.  Type `stack build` to build it, and `stack exec blog-tutorial-exe` to run it.

To test it out, try some basic curl commands:
```bash
curl 127.0.0.1:8080/users
curl 127.0.0.1:8080/posts
curl -d '{"email": "nikolatesla@hotmail.com", "password": "123abc"}' \
     -H "Content-type:Application/JSON" 127.0.0.1:8080/users
```

(continued in [Part 2](2016-08-31-servant-opaleye-part_2.html#disqus-thread))