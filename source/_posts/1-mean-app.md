---
layout: post
title: "1-Angular 5 MEAN app"
subtitle: "Setting up MEAN app with angular cli and express-generator"
date: 2018-03-04 13:37
author: "Andrei Sbitnev"
header-img: "/img/mean_code.png"
cdn: 'header-off'
tags:
	- MEAN
	- Angular 5
	- Node.js
	- Express
---
## Introduction

&nbsp; In this tutorial, we will build a base MEAN app using <b>Angular 5</b>, <b>MongoDb</b> and <b>Express.js</b>. The frontend and backend parts will be separated into different directories, and will be run as separate services.<br>
There are different possible ways to structure a MEAN app, and separating the frontend from backend has its own advantages:
<ul>
<li>The structure is simpler and easier to maintain. With time your app will grow, and the number of packages used in both parts will also grow.</li>
<li>Most probably different tools will be used for unit testing frontend and backend.</li>
<li>Easier to build the app using tools like angular cli and express-generator.</li>
</ul>
The base app can later be used to quickly start a new MEAN project, and will be used in further tutorials to build the <b>smart-house</b> project.

## Quick Start

To build the project you need the [angular cli](https://github.com/angular/angular-cli) and [express-generator](https://expressjs.com/en/starter/generator.html) installed on your operating system.<br>

You will also need a [mongoDb](https://www.mongodb.com/download-center#community) either installed on your system, or running inside a Docker container. Since we\`ll be Dockerizing the project in the next tutorial, we will use the container version.<br>

To run any docker container, [Docker](https://docs.docker.com/install/#time-based-release-schedule) must first be installed on your system. 
>Tip: Use a CE version

>Tip: If you\`re on a windows environment, be sure to run the command in powershell

To create and run a mongoDb container, just run the
```bash
$ sudo docker container run --name mongodb -d -p 127.0.0.1:27017:27017 mongo
```
>TIP: -d - means run detached, -p - port forwarding, mongodb - is the name of your container. You can then start and stop the container by running ``sudo docker container start mongodb`` or ``sudo docker container stop mongodb``.

## Setting up the project
Create a new directory <b>smart-house</b> and cd into it.

```bash
mkdir smart-house && cd smart-house
```

Start a new Angular 5 project using angular cli

```bash
ng new fe
```

Start a new express project using express-generator

```bash
express be
```

## Configuring backend
Open up the project in your favorite text editor and open the ~/smart-house/be/app.js file

We will be using the <b>be</b> app only as a REST api, so we don\`t need views or view engine. Remove the following lines from the file.

```javascript
// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');
```

And the views folder

```bash
rm -rf be/views
```

We also won\`t be serving any static files or a favicon with the <b>be</b> app, so remove the following lines from the ~/be/app.js file

```javascript
var favicon = require('serve-favicon');
...
app.use(express.static(path.join(__dirname, 'public')));
```

And the public folder

```bash
rm -rf be/public
```

We also won\`t be needing the index route, so let\`s remove it from the project. Remove the following lines from the from the ~/be/app.js file.

```javascript
var index = require('./routes/index');
...
app.use('/', index);
```
And the index.js file from the ~/be/routes directory

```bash
rm be/routes/index.js
```

We can also remove the unnecessary dependencies from the ~/be/package.json file.
```javascript
"serve-favicon": "~2.4.5"
...
"jade": "~1.11.0",
```
>Tip: Don`t forget to remove the comma from the end of the package.json file.

Now let\`s install all the dependencies. Just cd into the ~/smart-house/be folder and run 

```bash
npm install
```

Although it is possible to communicate with mongoDb without any additional packages, we will be using the Mongoose.js library for that purpose. So let\`s add it to the dependencies. Just cd into the ~/smart-house/be folder and run 

```bash
npm install --save mongoose
```

Now let\`s add mongoose to app.js
```javascript
var mongoose = require('mongoose');
```
And connect to the database.
```javascript
mongoose.connect('mongodb://localhost/test');
```
Since our frontend and backend will be running as separate services on different ports, or maybe even on different servers, we need to set appropriate headers, to allow frontend send requests to the backend.
```javascript
...
app.use(cookieParser());

app.use(function(req, res, next) {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader(
    "Access-Control-Allow-Headers",
    "Origin, X-Requested-With, Content-Type, Accept"
  );
  res.setHeader(
    "Access-Control-Allow-Methods",
    "POST, GET, PATCH, DELETE, OPTIONS"
  );
  next();
});

app.use('/users', users);
...
```
The final ~/be/app.js file should look like this
```javascript
var express = require('express');
var path = require('path');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var mongoose = require('mongoose');
var users = require('./routes/users');

var app = express();
mongoose.connect('mongodb://localhost/test');
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());

app.use(function(req, res, next) {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader(
    "Access-Control-Allow-Headers",
    "Origin, X-Requested-With, Content-Type, Accept"
  );
  res.setHeader(
    "Access-Control-Allow-Methods",
    "POST, GET, PATCH, DELETE, OPTIONS"
  );
  next();
});

app.use('/users', users);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;

```
## Simple server for the frontend
In this project build we are using angular cli, which is a simple and powerful tool for building and running angular applications in development. It comes with a preconfigured testing environment and a dev server, but it is not a good idea to use the dev server in production. Lets next setup a simple production server for angular.<br>
Go to ~/smart-house/fe and add a new file server.js

```javascript
var express = require('express');
var path = require('path');
var app = express();

app.use(express.static(path.join(__dirname, 'dist')));

app.get('*', function (req, res) {
    res.sendFile('index.html');
});

app.listen(4200, function () {
  console.log('fe running on port 4200');
});
```

In the code above we add a static directory, which will hold all of our static files and most importantly the bundle.js file. By default angular cli will create a bundle in the dist folder of your project.<br>

```javascript
app.use(express.static(path.join(__dirname, 'dist')));
```

We then setup a single endpoint with a wildcard *, which will always get called, no matter what is specified in the url.<br>

```javascript
app.get('*', function (req, res) {
    res.sendFile('index.html');
});
```

This is important, because all the routing and error handling in an angular app should be done on the frontend.<br>

Finally we add a server to listen on port 4200, which is the same port used to serve the application by angular dev server.<br>

```javascript
app.listen(4200, function () {
  console.log('fe running on port 4200');
});
```

Lets now modify the <b>fe</b> package.json file to use the newly created server. Go to  ~/smart-house/fe and modify the <b>scripts</b> object in the package.json file. 

```json
...
"scripts": {
    "ng": "ng",
    "start": "node server.js",
    "dev": "ng serve",
    "build": "ng build --prod",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e"
  },
...
```
To run the angular dev server use `npm run dev`. To run the app in production, first build it `npm run build`, and then run it `npm start`.

## Testing the MEAN app components working together.
The MEAN app is basically setup by now, but it would be great to test if angular, express and mongoDb are correctly set up and can communicate with each other.<br>
For this purpose we will add a <b>test</b> component to our angular app, which will have a form with a single input and a button. The form will be sent to the backend, and then saved to mongoDb in the <b>test</b> database. We will also fetch all the values from the <b>test</b> database and display them below the form as a list.<br>
We won\`t be going into further detail about angular or express at this point, because it\`s not the purpose of this tutorial. All the steps will be shown and described in upcoming tutorials, when we\`ll be actually building the app.

Go to ~/smart-house/fe directory and run the `ng generate component test` to add a new component to angular app. Below are listed all the files, that need to be added or changed for the test app to run correctly, and what they should contain.<br>

~/smart-house/fe/src/app/app.component.html

```html
<app-test></app-test>
```

~/smart-house/fe/src/app/test/test.component.html

```html
<div>
  <form #f="ngForm" (ngSubmit)="onSubmit(f)">
    <label for="text">Text</label>
    <input type="text" id="text" name="text" ngModel required>
    <button type="submit" [disabled]="!f.valid">Submit</button>
  </form>
  <ul *ngFor="let text of textArr">
    <li>{{text}}</li>
  </ul>
</div>
```

~/smart-house/fe/src/app/test/test.component.ts

```javascript
import { Component, OnInit, ViewChild } from '@angular/core';
import { NgForm } from '@angular/forms';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import 'rxjs/Rx';

@Component({
  selector: 'app-test',
  templateUrl: './test.component.html',
  styleUrls: ['./test.component.css']
})
export class TestComponent implements OnInit {
  @ViewChild('f') testForm: NgForm;
  public textArr = [];
  public httpOptions = {
    headers: new HttpHeaders({ 'Content-Type': 'application/json' })
  };

  getTestData() {
    this.http.get('http://localhost:3000/test')
      .subscribe(
        data => {
          this.textArr = data['obj'].map(val => val['text']);
        },
        error => {
          console.log(error);
        }
      );
  }

  sendTestData(text) {
    let body = JSON.stringify(text);
    this.http.post('http://localhost:3000/test', body, this.httpOptions)
      .subscribe(
        data => {
          this.testForm.resetForm();
          this.getTestData();
        },
        error => {
          console.log(error)
        }
      )
  }

  constructor(private http: HttpClient) {}
  onSubmit(form: NgForm) {
    this.sendTestData(form.value)
  }
  ngOnInit() {
    this.getTestData()
  }
}
```
~/smart-house/fe/src/app/app.module.ts

```javascript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { AppComponent } from './app.component';
import { TestComponent } from './test/test.component';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [
    AppComponent,
    TestComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
Add a new directory <b>models</b> in ~/smart-house/be, which will hold the mongoose.js models, and add a <b>test</b> model.<br>

~/smart-house/be/models/test.js

```javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var schema = new Schema({
    text: {type: String, required: true},
});

module.exports = mongoose.model('Test', schema);
```

Add a new routes file test.js

~/smart-house/be/routes/test.js

```javascript
var express = require('express');
var express = require('express');
var router = express.Router();
var Test = require('../models/test');

router.get('/', function(req, res, next) {
  Test.find()
        .exec(function (err, textArr) {
            if (err) {
                return res.status(500).json({
                    title: 'An error occurred',
                    error: err
                });
            }
            res.status(200).json({
                message: 'Success',
                obj: textArr
            });
        });
});

router.post('/', function(req, res, next) {
  var text = req.body.text;
  var test = new Test({text})
  test.save();
  res.status(200).json({
    message: 'Success',
  });
});

module.exports = router;

```
Add a new <b>test</b> route to be/app.js file
~/smart-house/be/app.js

```javascript
...
var users = require('./routes/users');
// load the test model
var test = require('./routes/test');
...
app.use('/users', users);
// add test router
app.use('/test', test);
...

```
## Starting the application
To test the application, first start the mongodb server
>Tip: If it is set up in the Docker container, as described above, just run `sudo docker container start mongodb`<br>

Start the backend server, by navigating inside the ~/smarthouse/be and running `npm start`<br>
Open up another terminal and start an angular app, by navigating inside the ~/smarthouse/fe and running `npm run dev`<br>
Now open the browser and go to [http://localhost:4200](http://localhost:4200), you should see a white page with one input field and a disabled "submit" button. Enter something in the input field and submit it to the server. The submitted text should appear below the form as a line in a list. You can now refresh the page, and the list should remain unchanged.
<img src="/img/finished.png">

## Git repository
The version of MEAN app built in this tutorial is available in a github repo, on the [1-mean-app-base](https://github.com/andreisbitnev/smart-house/tree/1-mean-app-base) branch. You can git clone it with
```bash
git clone -b 1-mean-app-base https://github.com/andreisbitnev/smart-house.git
```

## Next up
In the next tutorial we will be Dockerizing the base app, so go ahead and check it out [2-MEAN app with Docker](/2018/03/01/2-mean-docker/)