---
title: Implementing a flash middleware on express.js 3.x
date: 2014-01-22 21:20:25
tags: [JavaScript, NodeJs, ExpressJs]
---

[1]: http://www.pluralsight.com/training/Courses/TableOfContents/full-stack-nodejs
[2]: http://en.wikipedia.org/wiki/Post/Redirect/Get
[3]: https://github.com/visionmedia/express/wiki/Migrating-from-2.x-to-3.x

I’m currently following the Pluralsight training [Full Stack Node.js][1]. The course was release more than a year ago and it is based on express.js 2.x. 
I have chosen to build the course sample on express 3.x and because of this some features described on the course are no more applicable. 
The purpose of this blog post is to actually demonstrate how to migrate properly one of the them: “The Flashing”.

Simply put, the flashing consist of displaying a status message once an operation executed on a post has completed. 
The Flashing is particularly useful when using the [Post/Redirect/Get Pattern][2]. The all scenario can be described as:

1. The user submit a form with a POST action
2. The user form is process and a result status is generated such as “Error” or “Success”
3. The Post action completes with a redirect that will tell the client browser to GET another page.
4. The rendering of the get result should display the processing status

I found several questions on stackexchange explaining how to do that but none of them was clear enough for me to understand… 
So I digged a bit and I decide to share my findings here.

The original solution was using app.dynamicHelpers which is not applicable anymore. 
The [migration document from express 2.x to 3.x][3] just says replace with:

middleware and res.locals…

Fine… How am I suppose to do that? Well I believe the answer is incomplete.

You will use the middleware to add processing into the handling of your request and its response. 
After all this is why the middleware is there for. Then in the **middleware** you will attach a function that can be used in the route handler.
This function will use **res.locals** to attach something to your response that can be used on the rendering of the view. This is incomplete…  
or at least this how I understood it and it felt incomplete.

Our problem here is that we are using the Post/Redirect/Pattern and this means that all data attach to the res.locals vanish as soon as you do a redirect.
The redirect will instruct the browser to perform a GET and this will start the request from the processing of the HTTP request from the beginning.
There is no way to solve our issue just with a middleware and res.locals!!! We need a way to pass information from the POST and REDIRECT to the following GET. 
The only to do this is via a cookie or in the session. But those will then need to be cleaned once the status message has been displayed.

This is how I have achieved this. First I have implemented the middleware and it looks like this:

```javascript
"use strict";
 
var currentRes;
var currentReq;
module.exports = function(){
    return function(req, res, next) {
        //We save the response and request reference has we will need it later
        currentRes = res;
        currentReq = req;
        //We attach the flash function to be used from the route handler
        req.flash = _flash;
        //We read the status from the session if it is there and we remove it
        _flash();
        next();
    };
};
 
var _flash = function(type,message){
    //We need a session if there is none... Then we raise an exception
    if (currentReq.session === undefined) {
        throw Error('req.flash() requires sessions');
    }
    //This the usage from the route handler. This will store the status message in the POST processing
    if(type && message){
        currentReq.session.flash = {flashType:type,flashMessage:message};
    } else {
        //If no parameter are passed then we read the eventual value saved on the session and we attach it to res.locals for the rendering
        if (currentReq.session.flash){
            var flashObj = currentReq.session.flash;
            currentRes.locals.flashTypes = ['info','error'];
            currentRes.locals.flash = {};
            currentRes.locals.flash[flashObj.flashType] =flashObj.flashMessage;
            delete currentReq.session['flash'];
        }
    }
};

```

The middleware will need to be added to your app express instance as usual.

```javascript
var flash = require('./middleware/flash');
 
var app = module.exports= express();
 
app.configure(function(){
    app.set('port', process.env.PORT || 3000);
    app.set('views', path.join(__dirname, 'views'));
    app.set('view engine', 'jade');
    app.use(express.favicon());
    app.use(express.logger('dev'));
    app.use(express.json());
    app.use(express.urlencoded());
    app.use(express.methodOverride());
    app.use(express.cookieParser());
    app.use(express.session({
        secret:"mysupersecrethash",
        store: new RedisStore()
    }));
    app.use(flash());
    app.use(app.router);
    app.use(express.static(path.join(__dirname, 'public')));
 
    if('test' === app.get('env')){
        app.set('port',3001);
    }
 
    // development only
    if ('development' === app.get('env')) {
      app.use(express.errorHandler());
    }
});

```
Then from the route handler of the POST processing we have the following implementation. 
This is basically where we set the status of the processing.

```javascript
app.post('/sessions',function(req,res){
    if(('admin' === req.body.user) && '12345'===req.body.password){
        req.session.currentUser = req.body.user;
        req.flash('info',"You are logged in as "+req.session.currentUser);
        res.redirect('/login');
        return;
    }
    req.flash('error',"Those credentials were incorrect. Try Again");
    res.redirect('/login');
});

```
And finally we can use the status information that will have been attached to the res.locals from the jade template

```javascript
if (typeof(flash) !=='undefined')
    each flashType in flashTypes
        if flash[flashType]
            p.flash(class=flashType) #{flash[flashType]}

```
