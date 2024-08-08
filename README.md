# BLOGG


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= title %></title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <header>
        <h1><a href="/">My Blog</a></h1>
        <nav>
            <a href="/">Home</a>
            <a href="/posts/new">New Post</a>
        </nav>
    </header>
    <div class="content">
        <%= content %>
    </div>
</body>
</html>

<% include layout.ejs %>

<% content %>
<h2>Blog Posts</h2>
<ul>
    <% posts.forEach(function(post) { %>
        <li>
            <h3><a href="/posts/<%= post._id %>"><%= post.title %></a></h3>
            <p><%= post.content.substring(0, 100) %>...</p>
        </li>
    <% }) %>
</ul>
<% endcontent %>

<% include layout.ejs %>

<% content %>
<h2>Create a New Post</h2>
<form action="/posts" method="POST">
    <label for="title">Title:</label>
    <input type="text" id="title" name="title" required>

    <label for="content">Content:</label>
    <textarea id="content" name="content" rows="10" required></textarea>

    <input type="submit" value="Publish">
</form>
<% endcontent %>

body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    margin: 0;
    padding: 0;
}

header {
    background-color: #333;
    color: #fff;
    padding: 10px 0;
    text-align: center;
}

header h1 a {
    color: #fff;
    text-decoration: none;
}

nav {
    margin-top: 10px;
}

nav a {
    color: #fff;
    text-decoration: none;
    margin: 0 15px;
}

.content {
    padding: 20px;
    max-width: 800px;
    margin: 20px auto;
    background-color: #fff;
    box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}

h2 {
    margin-top: 0;
}

input[type="text"],
textarea {
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

input[type="submit"] {
    background-color: #28a745;
    color: white;
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

input[type="submit"]:hover {
    background-color: #218838;
}

ul {
    list-style-type: none;
    padding: 0;
}

ul li {
    margin-bottom: 20px;
}
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const app = express();

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/blogDB', { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('Connected to MongoDB'))
    .catch(err => console.error('Could not connect to MongoDB...', err));

// Middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static('public'));
app.set('view engine', 'ejs');

// Define a schema and model for posts
const postSchema = new mongoose.Schema({
    title: String,
    content: String,
});

const Post = mongoose.model('Post', postSchema);

// Routes
app.get('/', (req, res) => {
    Post.find({}, (err, posts) => {
        if (err) {
            console.log(err);
        } else {
            res.render('home', { title: 'Home', posts: posts });
        }
    });
});

app.get('/posts/new', (req, res) => {
    res.render('new-post', { title: 'New Post' });
});

app.post('/posts', (req, res) => {
    const { title, content } = req.body;

    const newPost = new Post({
        title,
        content,
    });

    newPost.save()
        .then(() => res.redirect('/'))
        .catch(err => res.status(400).send('Error: ' + err));
});

app.get('/posts/:id', (req, res) => {
    Post.findById(req.params.id, (err, post) => {
        if (err) {
            console.log(err);
        } else {
            res.render('post', { title: post.title, post: post });
        }
    });
});

// Start the server
const port = 3000;
app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});
