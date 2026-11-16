# 📸 Binary Upload Boom | Photo-Sharing Social Network

A full-stack photo-sharing social network. Users sign up, upload photos with a title and caption, like posts, and talk in threaded comment sections with replies, likes and dislikes, and profile pictures.

Built with Node.js, Express, MongoDB, Passport, Cloudinary, and EJS, following the MVC pattern. It started from the 100Devs social network starter, and I added the comment system and profile pictures.

---

## ✨ Features

### Accounts

- **Sign up and log in** with email and password (Passport local strategy)
- **Secure passwords**: hashed with bcrypt before they're saved
- **Form validation** with `validator`: valid email, 8+ character password, matching confirmation, and unique email and username
- **Sessions stored in MongoDB** with `connect-mongo`, so users stay logged in after a server restart
- **Profile pictures**: upload an avatar, auto-cropped to a 200×200 square centered on the face

### Posts

- **Upload photos** (`.jpg`, `.jpeg`, `.png`) with a title and caption. Images are stored on **Cloudinary**.
- **Profile page** with your own posts, and a **feed** with everyone's posts, newest first
- **Like posts**
- **Delete your posts**: removes the image from Cloudinary and the post's comments from the database

### Comments (my addition)

- **Threaded replies**: reply to any comment, nested to any depth
- **Like and dislike** each comment. Each user can react once, and clicking again removes the reaction.
- **Author-only delete**: deleting a comment also deletes every reply under it
- **Avatars and usernames** next to each comment
- **Jump back to your comment** after posting, replying, or reacting, using `#comment-<id>` links

---

## 🔍 How It Works

```
Browser ──▶ Express routes ──▶ controllers ──▶ Mongoose models ──▶ MongoDB
                                   │
                                   └──▶ Cloudinary (image uploads)
```

### Routes

| Method     | Route                            | What it does                           |
| ---------- | -------------------------------- | -------------------------------------- |
| GET        | `/`                              | Landing page                           |
| GET / POST | `/signup`                        | Sign-up form / create an account       |
| GET / POST | `/login`                         | Login form / log in                    |
| GET        | `/logout`                        | Log out                                |
| GET        | `/profile`                       | Your profile and posts                 |
| POST       | `/profile/avatar`                | Upload or replace your profile picture |
| GET        | `/feed`                          | All posts                              |
| GET        | `/post/:id`                      | A single post with its comment thread  |
| POST       | `/post/createPost`               | Upload a new post                      |
| PUT        | `/post/likePost/:id`             | Like a post                            |
| DELETE     | `/post/deletePost/:id`           | Delete a post                          |
| POST       | `/comment/createComment/:postId` | Add a comment                          |
| POST       | `/comment/reply/:commentId`      | Reply to a comment                     |
| PUT        | `/comment/like/:id`              | Like or un-like a comment              |
| PUT        | `/comment/dislike/:id`           | Dislike or un-dislike a comment        |
| DELETE     | `/comment/delete/:id`            | Delete your comment and its replies    |

HTML forms can only send GET and POST, so `method-override` turns `?_method=PUT` and `?_method=DELETE` into real PUT and DELETE requests.

### How comment threads work

Each comment stores a `parentComment`: `null` for a top-level comment, or the `_id` of the comment it replies to.

- **Showing a thread**: the post page loads all of a post's comments in one query. Then `buildCommentTree()` uses a `Map` to turn the flat list into a tree, where each comment has a `replies` array.
- **Deleting a thread**: `collectThreadIds()` walks down the tree one level at a time to find every reply under a comment, then deletes them all with one `deleteMany`.
- **Reactions**: `likes` and `dislikes` are arrays of user IDs, not plain counts. That way each user can only react once, and liking a comment removes your dislike.

### Data models

- **User**: `userName`, `email`, `password` (hashed)
- **Post**: `title`, `image`, `cloudinaryId`, `caption`, `likes`, `user`, `createdAt`
- **Comment**: `text` (up to 1,000 characters), `post`, `user`, `parentComment`, `likes`, `dislikes`, timestamps

---

## 🛠️ Tech Stack

- **Node.js** and **Express**
- **MongoDB** with **Mongoose**
- **Passport** (`passport-local`) and **bcrypt** for authentication
- **express-session** and **connect-mongo** for sessions
- **Cloudinary** and **Multer** for image uploads
- **EJS** templates with **Bootstrap 5** and **Font Awesome**
- **method-override**, **express-flash**, **validator**, **morgan**, and **dotenv**

---

## 📁 Project Structure

```
binary-upload-boom/
├── server.js               # App setup, middleware, and routes
├── config/
│   ├── database.js         # MongoDB connection
│   ├── passport.js         # Login strategy
│   └── .env                # Your environment variables (not committed)
├── controllers/
│   ├── auth.js             # Sign up, log in, log out
│   ├── home.js             # Landing page
│   ├── posts.js            # Profile, feed, posts, avatars, comment tree
│   └── comments.js         # Comments, replies, reactions, thread delete
├── middleware/
│   ├── auth.js             # ensureAuth: protects routes
│   ├── cloudinary.js       # Cloudinary setup
│   └── multer.js           # Image upload handling
├── models/                 # User, Post, Comment
├── routes/                 # main, posts, comments
├── views/                  # EJS pages and partials (header, footer, comment)
└── public/                 # CSS and images
```

---

## 🚀 Getting Started

You need [Node.js](https://nodejs.org/), a [MongoDB Atlas](https://www.mongodb.com/atlas) database, and a free [Cloudinary](https://cloudinary.com/) account.

### 1. Clone and install

```bash
git clone https://github.com/kareemahb/binary-upload-boom.git
cd binary-upload-boom
npm install
```

### 2. Add your environment variables

Create a `.env` file **inside the `config` folder** (`config/.env`):

```
PORT=2121
DB_STRING=your MongoDB connection string
CLOUD_NAME=your Cloudinary cloud name
API_KEY=your Cloudinary API key
API_SECRET=your Cloudinary API secret
SESSION_SECRET=a long random string
```

| Variable                              | What it is                                                 |
| ------------------------------------- | ---------------------------------------------------------- |
| `PORT`                                | Port to run on. Defaults to `2121`.                        |
| `DB_STRING`                           | Your MongoDB connection string (Atlas → Connect → Drivers) |
| `CLOUD_NAME`, `API_KEY`, `API_SECRET` | From your Cloudinary dashboard                             |
| `SESSION_SECRET`                      | Signs login sessions. Use your own random string.          |

### 3. Run it

```bash
npm start
```

Then open [http://localhost:2121](http://localhost:2121).

---

## 💡 What I Learned

- Organizing an Express app with the MVC pattern: routes, controllers, models, and views
- Modeling nested data with a self-referencing `parentComment` field
- Turning a flat list into a tree with a `Map`, so a whole thread loads in one query
- Storing reactions as arrays of user IDs to allow one reaction per user
- Checking ownership on the server before deleting
- Uploading, transforming, and deleting images with Cloudinary
- Protecting routes with middleware and keeping sessions in MongoDB

---

## 🙏 Credit

Started from the [100Devs](https://100devs.org/) social network starter (MIT License). The comment system (threads, replies, reactions, thread delete) and profile pictures are my additions.

---

## 👩‍💻 Created By

**Kareemah Bahaadur**
[GitHub](https://github.com/kareemahb)
