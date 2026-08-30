# mern-blog

*Part of [[Projects]] → [[Study and Practice]]*

**Repo:** [mern-blog](https://github.com/prakharDvedi/mern-blog)
**Visibility:** Public
**Language:** JavaScript

---

# 📝 BlogIT

**BlogIT** is a full-stack blog application built using the MERN stack (MongoDB, Express.js, React, Node.js). It empowers users to **create**, **read**, **update**, and **delete** blog posts with secure authentication, a responsive design, and a sleek UI.

---

## 🌟 Features

* 🔐 **User Authentication**: Secure login & registration with JWT (JSON Web Tokens)
* 🖠️ **CRUD Operations**: Seamless create, read, update, and delete functionality for blogs
* 📱 **Responsive Design**: Mobile-friendly UI built with React & styled using Tailwind CSS
* 🧠 **MongoDB Integration**: Persistent storage for users and posts
* 📡 **RESTful API**: Built using Express.js for smooth backend communication

---

## 🧰 Tech Stack

| Tech          | Description                       |
| ------------- | --------------------------------- |
| 🟢 MongoDB    | NoSQL database for user/blog data |
| ⚙️ Express.js | Backend routing and middleware    |
| ⚛️ React      | Dynamic, component-based UI       |
| 🜩 Node.js    | Server-side JavaScript runtime    |
| 🛡️ JWT       | Token-based secure authentication |
| 📦 Other      | Axios, bcrypt, dotenv, cors, etc. |

---

## 🚀 Prerequisites

Make sure you have the following installed:

* ✅ Node.js (v16 or later)
* ✅ MongoDB (local or MongoDB Atlas)
* ✅ npm or yarn

---

## 🥪 Installation

### 1. 🔁 Clone the Repository

```bash
git clone https://github.com/prakharDvedi/mern-blog.git
cd mern-blog
```

### 2. ⚙️ Backend Setup

```bash
cd server
npm install
```

📝 Create a `.env` file inside the `server/` folder:

```env
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret_key
PORT=5000
```

> Replace `your_mongodb_connection_string` and `your_jwt_secret_key` with real values.

### 3. 💻 Frontend Setup

```bash
cd ../client
npm install
```

---

## ▶️ Running the App

### Start Backend

```bash
cd server
npm start
```

### Start Frontend

```bash
cd client
npm start
```

* 🖙 Backend → `http://localhost:5000`
* 🖜 Frontend → `http://localhost:3000`

---

## 🛍️ Usage

1. Open your browser and visit: `http://localhost:3000`
2. Register a new account or log in.
3. Create, edit, or delete blog posts.
4. Read posts created by other users.

---

## 🗂️ Project Structure

```
mern-blog/
├── client/                 # React frontend
│   ├── public/             # Static assets
│   ├── src/                # Components, pages, styles
│   └── package.json        # Frontend dependencies
├── server/                 # Express backend
│   ├── models/             # Mongoose schemas
│   ├── routes/             # API endpoints
│   ├── controllers/        # Route logic
│   ├── middleware/         # Auth middleware
│   └── package.json        # Backend dependencies
├── .gitignore              # Git ignored files
└── README.md               # Documentation
```

---

## 🤝 Contributing

Contributions are welcome! 🚀

```bash
1. Fork the repo
2. git checkout -b feature/your-feature
3. Make changes and commit (git commit -m "feat: your feature")
4. Push to origin (git push origin feature/your-feature)
5. Open a Pull Request
```

---

## 📜 License

Licensed under the [MIT License](LICENSE).

---

## 🙏 Acknowledgements

* Inspired by several MERN stack tutorials and projects
* Thanks to the open-source community for amazing tools and libraries

---

## ✨ Live Deployment (Optional)

> Add your live demo links here if deployed on **Vercel**, **Render**, or **Netlify**

---
