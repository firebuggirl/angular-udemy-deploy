* Angular app is tied into Node app via `views/index.hbs`

* Open 4 terminal windows and run:

 - ` mongod `

 - ` mongo `

 - ` npm run build `

 - ` npm start `

 * Visit ` localhost:3000 `

 * routes/messages.js:

  -  ` obj: result ` //line 64...this ties messages.service.ts from Angular/front end in with backend DB


* Connect front end to back end DB:

  1. add/edit/delete message (in the front message array)

  2. use message service to connect backend and then post/patch/delete the data (MessageListComponent uses this.messageService.getMessages connect to
  router.get, which gets messages from the data in the DB and uses this.messages = messages;  to update the front data and then also in MessageListComponent use*ngFor="let message of messages to let us see the updated front data from backend data in the front end.


## Coding the sign in

* Start with backend in `routes/user.js`

* Create JWT token

  - ` npm i --save jsonwebtoken `

# More info on JWT:

  - https://auth0.com/learn/json-web-tokens/

  - https://scotch.io/tutorials/the-anatomy-of-a-json-web-token

  - https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage

* Connect back end to front end:

 - Go to `auth/auth.service.ts`

    - create signin method

 - Go to `auth/signin.component.ts`

    - create new User in onSubmit() with `http://localhost:3000/user/signin` leading to `signin` path pointing to `signin` path in `routes/user.js`

    - ` import { User } from './user.model'; `

    - ` import { Router } from '@angular/router'; `

    - ` constructor(private authService: AuthService, private router: Router){} `

    - ` import { AuthService } from "./auth.service"; `

    - ` npm run build `

    - ` `

# Implement Log Out button

   - in auth/auth.service.ts:

     - add logout() method

  - in auth/logout-component.ts:

    ` import { AuthService } from "./auth.service"; `

    - add authService to constructor + import Router....

    - Call authService function from authService:

     ` this.authService.logout() `

# Show/hide buttons/tabs based on whether user is logged in or not

  - in auth/auth.service.ts:

    - add isLoggedIn() method

  - in auth/authentication.component.ts:

      - inject authService via constructor function + ` import { AuthService } from "./auth.service"; `

      - add ` isLoggedIn() ` method

      - add `*ngIf` directives to links in template


## Backend Route Protection w/JWT

* in routes/messages.js:

  - add `jwt.verify` via `router.use` ,which will be reached on each request ...use `secret` to validate incoming token

  - `secret` has to match the `secret` created when secret was created via `routes/user.js` via token variable

  - ` var jwt = require('jsonwebtoken'); `


## Connect Users with messages:

  * remove current messages from DB:

  ` db.messages.remove({}) `


  * In `routes/messages.js `

    - retrieve user from token in `routes/user.js`

    - use `decode()`
      method  in `router.post`

    - pass decoded object to user field in `routes/user.js` via ` User.findById `

    - create `message.save(){}` function and push messages to array to messages on `User`

    - import User model:

    ` var User = require('../models/user'); `

  * In `models/message.js`:

    * In order to delete a message, need to delete from array of messages...also need to make sure that UserId is also deleted from messages array

    - set up `schema.post(){}` function to listen for `remove` action and pass `message.user` to `UserId`..so whenever a message is deleted, the message array on the User is updated to prevent any redundant data in DB

## Send Requests with a Token

  * Add a token for creating &/or sending a message and for updating/deleting

  * In `messages/message.service.ts`:

      - via `addMessage(message: Message)` method check 1st to see if there is a token in local storage via ` const token = localStorage.getItem('token') `

      - set up query string to append to end of url if token exists

      - add `token` to `updateMessage()` and append it AFTER `id`, then repeat with `deleteMessage()`

      - `recompile` ie., ` npm run build `

      NOTE: returns error in shell:

        `RangeError: Maximum call stack size exceeded`

        - https://www.udemy.com/angular-2-and-nodejs-the-practical-guide/learn/v4/questions/2829784

        - change `user.messages.push(result);` in `routes/messages.js` to :

            ` user.messages.push(
               mongoose.Types.ObjectId(result._id)
              ); `

              + import mongoose

        - restart `mongo`

          ` db.messages.find() `//check to see if user ID  was added to messages collection

          ` db.users.find() `//check to see if array of messages + messageId was added to User collection


## Handle User Authorization

   - ensure that only users that create a post are able to delete/edit messages

   * `routes/messages.js`

     - need to fetch user in patch function + check to see if this is user that created message

     - add `token` to `patch` route/method

     - check to see if `message.user` is !same as the user stored in token in patch route

     - implement/repeat same with delete route/method

     - restart server, test by creating new user(s) and creating/editing messages, etc....


## Passing the User Object with messages

  * Conditionally show buttons + show correct user:

    - routes/messages.js:

      - adjust `get` route:

        - add `.populate()` method ....a mongoose method which allows to expand data that is being retrieved
        ..by default would only retrieve the message object, with `.populate()` can create reference to `User` model..this is tied in/reference in `models/message.js` via `ref: 'User'` key/value pair, which is a connection to another object in DB
        ..can now tell Mongoose to expand object so that user field onot only holds user id, but the complete user object with all of its fields


## Front End Auth check

   * Fetch complete user in frontend via messages.service.ts `getMessages()`:

    - get user firstName in `let message of messages`
      for loop

    - populate `const message` with correct data in `addMessage()` method





    * Create new message + test to see if username is attached to message

## Hide/show buttons

  * In `message-component.ts`:

     - add `belongsToUser()` method

     NOTE: Important validation always happens on the backend, validation on the front end is strictly for the user experience

  * In `message-component.html`:

    - add ` *ngIf="belongsToUser()" `..ie., show buttons based on user state

    - recompile + reload


## NOTE from instructor:

  - if someone steals your Token, they may access your API. That's why tokens should be short-living and not be stored for long time durations. It's of course also a good idea to send HTTPS requests only. Your token can then only be stolen if the attacker somehow has access to your computer.

  * IP whitelisting also add a layer of security


## Error Handling

    * create new `errors directory`  in `app`

      -  create `error.service.ts`, ` error.model.ts`(to define how error should look on front end), `error.component.ts`, & `error.component.html`

      * `error.component.html`:

        - set up Bootstrap modal (HTML only)

        - create `.backdrop` class to show/hide conditionally

      * `error.component.ts`:

        - create property `error` of type `Error` + `display = none`

        - create `onErrorHandled()` method

      * `error.component.html`

        - add ngStyle directive w/ property binding to `.backdrop` to display...also add to `.modal`

        - Use string interpolation to output error title and message:

          - `{{ error?.title }}` & `{{ error?.message }}`//'?' helps to avoid errors

        - configure close button:

          - ` (click)="onErrorHandled()" `

      * `app.component.html`:

          - add/append ` <app-error></app-error> `

      * `app.module.ts`:

        -  ` import { ErrorComponent } ....
             import { ErrorService } ....`

             + add `ErrorComponent` to declarations & `ErrorService` to providers


## Set up errorService to emit and handle errors

* in `errors/error.service.ts`:

  - add ` export class ErrorService `

  - add ` import { EventEmitter } `

  - import ` import { Error } `

  - extract `error.title` from each error that is set up via the backend in `routes/messages.js` which always has an error object as well as a title object

  -   - extract `error.message` from each error that is set up via the backend ..........

  - now we are able to subscribe to `handleError` event in other places throughout app

* in `app.module.ts`:

  - add `errorService` to list array of providers

  - import ` import { ErrorService } `//enables application wide use

* in `errors/error.component.ts`:

   - add `implements OnInit`...listen to emitted events/errors

   - import `OnInit`

   - inject `ErrorService` in constructor

   - implement `ngOnInit()`..subscribe to errors/`ErrorService`

* in `messages/message.service.ts`:

   - emit/submit error(s):

   - inject `errorService` via the constructor...possible because `@Injectable()` has been added

   - import `errorService`

   - add `errorService` to `catch()`
      NOTE; `catch` function allows for injection of custom error code before continuing on with default error flow via `return Observable`

   - copy paste same `.catch()` referenced above and use for every catch method that follows


* in `auth/auth.service.ts`:

  - import + inject `errorService` via the constructor

  - copy paste same `.catch()` referenced above and use for every catch method that follows

* in `erros/error.component.html`:

  - remove `fade` class

* refresh and test errorService ..trying submitting message w/out auth


## App Optimizations, Lazy Loading and Deployment

* Lazy loading = load those modules/services on demand. ..decreases the startup time.

* Creating a message module:

  - in `assets/app/messages`:

     - create new file `message.module.ts`

     - import `import { NgModule }`

     - export class `Message`

     - declare all components which are only used w/ `Message` component

     - import `  
       MessagesComponent,
       MessageListComponent,
       MessageComponent,
       MessageInputComponent ` in declaration instead of in app.module.ts

https://www.udemy.com/angular-2-and-nodejs-the-practical-guide/learn/v4/t/lecture/5867130?start=0

    - import:

      - add `CommonModule` to `imports` array

      - import `import { NgModule } from '@angular/common';`

      - add `FormsModule` to `imports` array

      - import `import { FormsModule } from '@angular/forms';`

      - add `providers` array with `MessageService`

      - `import { MessageService } from './message.service';`

      ` import { CommonModule } from '@angular/common';
        import { FormsModule } from '@angular/forms';
        import { MessagesComponent } from "./messages.component";
        import { MessageListComponent } from "./message-list.component";
        import { MessageComponent } from "./message.component";
        import { MessageInputComponent } from "./message-input.component"; `




  - in `app.module.ts`:

    - `[AuthService, ErrorService]` = global services so do not modularize them, however `messageService` in `app/app.component.ts` is only used within messageService, so remove `messageService` from `app/app.component.ts` + strip out all `message-related components` from `app.module.ts` and instead outsource service and instead contain them within `messages.module.ts`

    - delete `FormsModule` from imports

    - add new import `MessageModule`

    - `import { MessageModule } from "./messages/message.module";`

## Using an Auth Module and Lazy Loading

  https://www.udemy.com/angular-2-and-nodejs-the-practical-guide/learn/v4/t/lecture/5867130?start=0

  - Only load Auth components if they are needed

  - in `assets/app/auth`:

      - create `auth.module.ts`

      - add component that are only used for auth:

        ` `

  * need to change routes for `auth` to enable lazy Loading

    - rename `auth/auth.routes.ts` to `auth/auth.routing.ts` + import `RouterModule` + `export const authRouting` w/ `forChild(AUTH_ROUTES)` (means 'part' of application, not 'entire' app...) method + register routes that are only used for auth purposes

    - in `app.routing.ts` use `forRoot()` method to register root routes fir entire application

        - remove `import { AUTH_ROUTES } from "./auth/auth.routes";`

        - change `children: AUTH_ROUTES ` to `loadChildren: './auth/auth.module#AuthModule' `//tells Angular to lazy load....

        - also need to specify name of class that is to be imported by appending #AuthModule (as seen above)

    - in `auth/auth.module.ts` import `authRouting`

  * Test lazy loading:

     - reload app, open dev tools, go to network tab, disable cache, reload app via `localhost:3000/messages`

     - click on `authentication` tab (while watching network tab) to see associated chunk/file that has now been downloaded

## Compiling the Application Ahead of Time (AoT)

   * Build for production:

    ` npm run build:prod `//script in package.json that uses 'offline compilation'

    - will now notice in network tab that file size is much smaller , g-zipping app will compress app even more

 * Dive deep into Angular 2 Modules:
  https://angular.io/docs/ts/latest/guide/ngmodule.html


## Deploy

   * Set up MongoLab to host MongoDB:

    https://mlab.com/home?newLogin=1

    - create new DB + create DB name

    - choose provider

    - choose single node + sandbox

    - click on db, then on `users`

         - create new DB User w/ name and password


    - copy DB URI string and enter it in `app.js` + set up `.env` variables to hide credentials

* Deploy to Heroku:

  - add `variables.env` to `.gitigore` BEFORE deploying

  - create new app in dashboard & name it

  - ` heroku login `

  -  Create repo on Github
      ` git init `
      ` git add . `
      ` git commit -m "first commit" `
      ` git remote add origin ....... `
      ` git push -u origin master `

  - initialize existing repo on Heroku from project   directory:

    - ` heroku git:remote -a angular-udemy-deployment `
