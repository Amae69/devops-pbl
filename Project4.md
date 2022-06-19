# PROJECT 4: MEAN STACK IMPLEMENTATION
##  **Task** : Implementing a simple Book Register web form using **MEAN** stack.
---

- ### **Step 1: Install NodeJs**

  - Update ubuntu

    `sudo apt update`

  - Upgrade ubuntu

    `sudo apt upgrade`

  - Add certificates

    `sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
    curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

  - Install NodeJS

    `sudo apt install -y nodejs`

- ### **Step 2: Install MongoDB**    

  `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

  `echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

  - Install MongoDB

    `sudo apt install -y mongodb`

  - Start The server

    `sudo service mongodb start`

  - Verify that the service is up and running

    `sudo systemctl status mongodb`

    ![mongodb running](https://user-images.githubusercontent.com/107736487/174503581-7dac05ef-39f5-4468-838f-18edafe60444.PNG)

  - Install npm – Node package manager.

    `sudo apt install -y npm`

  - Verify npm installation with the command below

    `npm -v`   

    ![npm version](https://user-images.githubusercontent.com/107736487/174503590-14096980-bd72-46c2-bf23-4681f62ef7ed.PNG)

  - Install body-parser package

    I'll need **‘body-parser’** package to help me process JSON files passed in requests to the server.

    `sudo npm install body-parser`

  - Create a folder named **‘Books’** and go into **Books** directory

    `mkdir Books && cd Books`

  - In the Books directory, Initialize npm project

    `npm init`  

  - Using vi editor, i'll add a file named **server.js** in the Book directory.

    `vi server.js`

  - Copy and paste the web server code below into the server.js file.
    ```
    var express = require('express');
    var bodyParser = require('body-parser');
    var app = express();
    app.use(express.static(__dirname + '/public'));
    app.use(bodyParser.json());
    require('./apps/routes')(app);
    app.set('port', 3300);
    app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
    });
    ```
- ### **Step 3: Install Express and set up routes to the server**

  - I'll use **Express** to pass book information to and from my **MongoDB** database.

  - I'll also use **Mongoose** to establish a schema for the database to store data of my book register.

    `sudo npm install express mongoose`

  - In **‘Books’** folder, create a folder named **apps** and cd into the **apps** folder

    `mkdir apps && cd apps`

  - In apps directory, create a file named **routes.js** and copy a code into it using vi editor

    `vi routes.js`

  - Copy and paste the code below into routes.js
    ```
    var Book = require('./models/book');
    module.exports = function(app) {
    app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
    }); 
    app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
     });
    });
    app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
     });
    });
    var path = require('path');
    app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
    });
    };
    ```
  
  - In the **‘apps’** folder, create a folder named **models** and cd into the **models** folder

    `mkdir models && cd models`

  - In **models** directory, create a file named **book.js** and copy a code into it using vi editor

    `vi book.js`

  - Copy and paste the code below into **‘book.js’**
    ```
    var mongoose = require('mongoose');
    var dbHost = 'mongodb://localhost:27017/test';
    mongoose.connect(dbHost);
    mongoose.connection;
    mongoose.set('debug', true);
    var bookSchema = mongoose.Schema( {
    name: String,
    isbn: {type: String, index: true},
    author: String,
    pages: Number
    });
    var Book = mongoose.model('Book', bookSchema);
    module.exports = mongoose.model('Book', bookSchema);  
    ```
- ### Step 4 – Access the routes with AngularJS
  I'll use **AngularJS** to connect my web page with **Express** and perform actions on my book register.

  - Change the directory back to **‘Books’**

    `cd ../..`

  - Create a folder named **public** and cd into the folder.

     `mkdir public && cd public` 

  - In **public** directory, create a file named **script.js** and copy a code into it using vi editor

    `vi script.js`

  - Copy and paste the code below (controller configuration defined) into the **‘script.js’** file
    ```      
    var app = angular.module('myApp', []);
    app.controller('myCtrl', function($scope, $http) {
    $http( {
    method: 'GET',
    url: '/book'
    }).then(function successCallback(response) {
    $scope.books = response.data;
    }, function errorCallback(response) {
    console.log('Error: ' + response);
    });
    $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
    };
    $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
       console.log('Error: ' + response);
    });
    };
    });
    ``` 
  - In **public** folder, create a file named **index.html** and copy a code into it using vi editor 

    `vi index.html`

  - Copy and paste the code below into **index.html** file. 
    
        <!doctype html>
        <html ng-app="myApp" ng-controller="myCtrl">
        <head>
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
        <script src="script.js"></script>
        </head>
        <body>
        <div>
        <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
        </table>
        <button ng-click="add_book()">Add</button>
        </div>
        <hr>
        <div>
        <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
        </table>
        </div>
        </body>
        </html>   
    
   - Change the directory back to Books

     `cd ..`

   - Start the server by running this command:

     `node server.js`

   - My server is now up and running on port 3300

     ![server running on port 3300](https://user-images.githubusercontent.com/107736487/174503789-2304c9f1-a984-4c2a-8112-2ae47a73b989.PNG)

   - To access my server on the internet, I'll need to open TCP port 3300 in my AWS Web Console for my EC2 Instance 

     ![port 3300 on ec2](https://user-images.githubusercontent.com/107736487/174503722-31c9ae02-00bd-4c9f-a241-077f957d2b6f.PNG)

   - Now i can access my Book Register web application from the Internet with a browser using my Public IP address or Public DNS name.

     ![result 1](https://user-images.githubusercontent.com/107736487/174503742-712b1e81-afa4-41ae-8351-7ea619c6f601.PNG)
   
     ![result 2](https://user-images.githubusercontent.com/107736487/174503762-fe1c8457-ae6a-4231-94af-1864be154de6.PNG)

- ## End of project...   
