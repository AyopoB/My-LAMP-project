# MEAN WEB STACK IMPLEMENTATION ON AMAZON AWS

As part of my DevOps learning journey, I implemented a MEAN stack (MongoDB, Express, Angular, Node.js) application for a simple Book Register web form. This project was deployed on an AWS EC2 instance, allowing me to explore integrating modern web technologies with cloud infrastructure. The purpose of this exercise was to gain practical experience in deploying a full-stack application, managing server-side and database functionalities, and creating a seamless user interface.

### Tools and Technologies Used
- MongoDB: Document database for storing book records.
- Express: Web framework for Node.js, used to build the backend API.
- Angular: Frontend framework for creating a dynamic user interface.
- Node.js: JavaScript runtime for executing the backend code.
- AWS EC2: Virtual server for hosting the application.

## STEPS INVOLVED
- Step 0 - [Preparing the  Prerequisites](preparinng-the-prerequisites)
- Step 1 - [Install NodeJs](install-nodejs)
- Step 2 - [Install MongoDB](install-mongodb)
- Step 3 - [Install Express and Setup routes to the server](install-express-and-setup-routes-to-the-server)
- Step 4 - [Access the routes with AngularJs](access-the-routes-with-angurlarjs)

## Step 0 - Preparing the Prerequisites
- AWS Account: Required to set up the Ubuntu server.
- Knowledge requirements: Familiarity with HTML, CSS, and Javascript.

**Set up Ubuntu Instance**

On AWS, Launch an EC2 with Ubuntu LTS 24, save the key pair to your device, or use an existing key pair if available.
![](img/launch%20instance.png)


SSH into the instance.
```bash
ssh -i <your-key.pem> ubuntu@<your-ec2-public-ip>
```



---

## Step 1 - Install NodeJs

1. Update and Upgrade Ubuntu
```bash
sudo apt update
sudo apt upgrade
```
2. Add certificates and Node.js repository:

```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

![](img/install-certs.png)

3. Install nodeJs:

```bash
sudo apt install -y nodejs
```
The above commands will install Node.js and npm (Node Package Manager).

![](img/install-nodejs.png)

---

## Step 2 -  Install MongoDB
1. Download the GPG key for MongoDB:

```bash
 sudo apt-get install -y gnupg curl 
 curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \ sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \ --dearmor
 ```
![](img/1st%20step%20for%20mongo.png)


2. Add the MongoDB repository to your system's package sources list:

 ```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```
![](img/2nd%20step%20for%20mongo.png)

3. Install MongoDB
```bash
sudo apt-get install -y mongodb-org
```

- Running the above command failed to install MongoDB, due to an outdated package list.
![](img/mongo%20unable-to-install.png)

- To fix the above issue, simply run the command below to update the package list :
```bash 
sudo apt update
```

- Then the  MongoDB install command again
![](img/mongo%20install.png)

4. Start and enable the MongoDb service with the following commands:
```bash
sudo systemctl start mongod
```
```bash
sudo systemctl enable mongod
```
```bash
sudo systemctl status mongod
```

![](img/start%20and%20enable%20Mongodb.png)

5. Install Body parser, which is used to handle form submissions and JSON data sent in POST requests:
```bash
sudo npm install body-parser
```
![](img/body-parser.png)
6. Create a new folder and name it 'Books', change the directory to the Books directory:

```bash
mkdir Books && cd Books
```

7. In the Books directory, Initialize the npm project:

```bash
npm init
```
![](img/npm%20init.png)

8. Add a new file named server.js :
```bash
vi server.js
```
9. Add the code below to server.js:
```js
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/test', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
  console.log(`Server up: http://localhost:${PORT}`);
});
```

---

##  Step 3 - Install Express and Setup routes to the server
Express will be used to pass book information to and from our MongoDB database.

Mongoose will be used to establish a schema for the database to store data of our book register.

1. Install Express:
```bash
sudo npm install express mongoose
```

![](img/express-install%20actuall.png)


2. In the Books directory create a folder named apps:
```bash
mkdir apps && cd apps
```
3. In apps folder, create a new file called routes.js:

```bash
vi route.js
```

4. Add the code below to routes.js:

```js
const Book = require('./models/book'); // Import the Book model
const path = require('path'); // Import path module for serving static files

module.exports = function(app) {
    
    // GET request to fetch all books
    app.get('/book', async (req, res) => {
        try {
            const books = await Book.find(); // Fetch all books from the database
            res.json(books); // Send the books as a JSON response
        } catch (err) {
            res.status(500).json({ message: 'Error fetching books', error: err.message }); // Handle errors
        }
    });

    // POST request to add a new book
    app.post('/book', async (req, res) => {
        try {
            const book = new Book({
                name: req.body.name, // Book name from request body
                isbn: req.body.isbn, // ISBN from request body
                author: req.body.author, // Author from request body
                pages: req.body.pages // Pages from request body
            });
            const savedBook = await book.save(); // Save the new book to the database
            res.status(201).json({ message: 'Successfully added book', book: savedBook }); // Send success response
        } catch (err) {
            res.status(400).json({ message: 'Error adding book', error: err.message }); // Handle errors
        }
    });

    // DELETE request to delete a book by its ISBN
    app.delete('/book/:isbn', async (req, res) => {
        try {
            const result = await Book.findOneAndDelete({ isbn: req.params.isbn }); // Find and delete book by ISBN
            if (!result) {
                return res.status(404).json({ message: 'Book not found' }); // Handle case where book is not found
            }
            res.json({ message: 'Successfully deleted the book', book: result }); // Send success response
        } catch (err) {
            res.status(500).json({ message: 'Error deleting book', error: err.message }); // Handle errors
        }
    });

    // Catch-all route to serve the frontend application (SPA)
    app.get('*', (req, res) => {
        res.sendFile(path.join(__dirname, '../public', 'index.html')); // Serve index.html for any non-API routes
    });
};
```
5. In the apps folder, create a new folder called models:
```bash
mkdir models && cd models
```
6. Create a new file called book.js
```bash
vi book.js
```
7. Add the code below to book.js

```js
const mongoose = require('mongoose'); // Import mongoose for MongoDB interaction

// Define the book schema
const bookSchema = new mongoose.Schema({
    name: { type: String, required: true }, // Book name is required
    isbn: { type: String, required: true, unique: true, index: true }, // ISBN is required, unique, and indexed
    author: { type: String, required: true }, // Author name is required
    pages: { type: Number, required: true, min: 1 } // Number of pages is required and must be at least 1
}, { 
    timestamps: true // Automatically add createdAt and updatedAt fields
});

// Export the Book model based on the schema
module.exports = mongoose.model('Book', bookSchema);
```


---

## Access the routes with AngularJs

AngularJS provides a web framework for creating dynamic views in your web applications.

1. Change the directory back to the 'Books' folder, and create a new folder called public:
```bash
mkdir public && cd public
```
2. In the public folder add a file named script.js:
```bash
vi script.js
```
3. Add the code below to script.js:
```js
angular.module('myApp', []) // Define an AngularJS module
.controller('myCtrl', function($scope, $http) { // Define a controller

    // Function to fetch all books
    function fetchBooks() {
        $http.get('/book')
        .then(response => {
            $scope.books = response.data; // Store fetched books in $scope
        })
        .catch(error => {
            console.error('Error fetching books:', error); // Handle fetch errors
        });
    }

    fetchBooks(); // Initial fetch of books when the controller loads

    // Function to delete a book by its ISBN
    $scope.del_book = function(book) {
        $http.delete(`/book/${book.isbn}`)
        .then(() => {
            fetchBooks(); // Refresh book list after deletion
        })
        .catch(error => {
            console.error('Error deleting book:', error); // Handle delete errors
        });
    };

    // Function to add a new book
    $scope.add_book = function() {
        const newBook = { 
            name: $scope.Name, 
            isbn: $scope.Isbn, 
            author: $scope.Author, 
            pages: $scope.Pages 
        }; // Prepare new book object
        
        $http.post('/book', newBook)
        .then(() => {
            fetchBooks(); // Refresh book list after adding
            // Clear form fields
            $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = ''; 
        })
        .catch(error => {
            console.error('Error adding book:', error); // Handle add errors
        });
    };

});
```
4. In the public folder, create a new file called index.html:

```bash
vi index.html
```
5. Add the code below to the index.html file:
```html
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl"> <!-- Define AngularJS app and controller -->
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Book Management</title>

    <!-- AngularJS Library -->
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
    <!-- External JavaScript file (script.js) -->
    <script src="script.js"></script>

    <!-- Basic styling for the page -->
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }

        /* Style for the form and its elements */
        form {
            background-color: #f9f9f9;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            max-width: 400px;
            margin-bottom: 20px;
        }
        h1, h2 {
            color: #333;
        }
        table { 
            border-collapse: collapse; 
            width: 100%; 
        }
        th, td { 
            border: 1px solid #ddd; 
            padding: 8px; 
            text-align: left; 
        }
        th { 
            background-color: #f2f2f2; 
        }
        input[type="text"], input[type="number"] { 
            width: 100%; 
            padding: 10px; 
            margin: 5px 0;
            box-sizing: border-box;
            border: 1px solid #ccc; 
            border-radius: 5px;
        }
        button { 
            margin-top: 10px; 
            padding: 10px 15px; 
            background-color: #4CAF50; 
            color: white; 
            border: none; 
            border-radius: 5px; 
            cursor: pointer; 
        }
        button:hover { 
            background-color: #45a049; 
        }
    </style>
</head>
<body>
    <h1>Book Management</h1>

    <!-- Form to add a new book -->
    <h2>Add New Book</h2>
    <form ng-submit="add_book()"> <!-- Calls add_book() on form submission -->
        <table>
            <tr>
                <td>Name:</td>
                <td><input type="text" ng-model="Name" required></td> <!-- Two-way binding for book name -->
            </tr>
            <tr>
                <td>ISBN:</td>
                <td><input type="text" ng-model="Isbn" required></td> <!-- Two-way binding for book ISBN -->
            </tr>
            <tr>
                <td>Author:</td>
                <td><input type="text" ng-model="Author" required></td> <!-- Two-way binding for book author -->
            </tr>
            <tr>
                <td>Pages:</td>
                <td><input type="number" ng-model="Pages" required></td> <!-- Two-way binding for book pages -->
            </tr>
        </table>
        <button type="submit">Add Book</button> <!-- Button to submit form -->
    </form>

    <!-- Display list of books -->
    <h2>Book List</h2>
    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th>ISBN</th>
                <th>Author</th>
                <th>Pages</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            <tr ng-repeat="book in books"> <!-- Loop through books and display each -->
                <td>{{book.name}}</td> <!-- Display book name -->
                <td>{{book.isbn}}</td> <!-- Display book ISBN -->
                <td>{{book.author}}</td> <!-- Display book author -->
                <td>{{book.pages}}</td> <!-- Display book pages -->
                <td><button ng-click="del_book(book)">Delete</button></td> <!-- Button to delete book -->
            </tr>
        </tbody>
    </table>
</body>
</html>

```
6. Change the directory back to Books:
```bash
cd ..
```

7. Start the server by running the command below:

```bash
node server.js
```

![](img/node%20serverjs.png)


8. To access the web app from our public IP on our browser, edit the inbound rules on the AWS security group for the current instance to include port 3300, as shown below:

![](img/change%20inbound%20rules.png)

Finally, the web app can be accessed at <public-ip>:3300:
![](img/web%20final%20answer...png)

![](img/web%20final%20answer2...png)



---

### Challenges and Obstacles Overcome

- **MongoDB Installation Issues**: During the installation of MongoDB, I encountered an issue where the database wouldn't install properly due to an outdated package list. Running `sudo apt update` resolved the issue, allowing MongoDB to be successfully installed. This experience highlighted the importance of ensuring that system packages are up-to-date before attempting software installations.

### Lessons Learned

1. **Keep System Packages Updated**: Encountering the MongoDB installation issue due to an outdated package list emphasized the need to regularly update system packages. Running `sudo apt update` is an essential step before installing new software to avoid potential compatibility issues.
   
2. **Systematic Troubleshooting**: When facing an error, I learned to approach troubleshooting systematically by checking for common issues like package updates, ensuring service statuses are active, and referring to documentation to quickly resolve problems.

3. **Cloud Deployment Understanding**: Deploying a full MEAN stack application on AWS EC2 gave me valuable insights into managing server-side applications in a cloud environment. It was essential to configure security groups correctly and ensure all necessary ports were open for proper access to the application.

By completing this project, I gained a deeper understanding of full-stack development and deployment on AWS, helping me further my DevOps knowledge.

---

