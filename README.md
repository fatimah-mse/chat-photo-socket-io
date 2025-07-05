# 📸 Photo-Chat - Real-time Chat App

**Photo-Chat** is a real-time chat application built using **Node.js**, **Express**, and **React**. It enables users to send text messages and images instantly to each other without refreshing the page. This is made possible through **Socket.io**, which provides real-time, bi-directional communication between the client and server.

---

## 🔌 What is Socket.io?

**Socket.io** is a powerful JavaScript library that enables real-time, event-based communication between the browser (client) and the server.

Unlike traditional HTTP requests, Socket.io keeps an open connection using WebSockets, allowing instant data exchange without the need to reload or poll the server repeatedly.

---

## ⚙️ How is Socket.io used in Photo-Chat?

The app uses Socket.io to broadcast messages and images to all users connected to the application in real time.

## 🧑‍💻 Username System

When a user opens the app, a username is automatically assigned or restored from previous sessions:

```js
const [currentUser, setCurrentUser] = useState(
  localStorage.getItem('username') || `User${Math.floor(Math.random() * 1000)}`
);
```

### 1⃣ Client Connection

```js
io.on('connection', async (socket) => {
    console.log('New client connected');})
```

Every time a user opens the app (or a new browser tab), a new connection is established via WebSocket. This creates a live communication channel between the server (Node.js) and the client (React).

---

### 2⃣ Sending Previous Messages on Connect

```js
const messages = await getMessages();
socket.emit('message_history', messages);
```

* When a new client connects, the server retrieves the last 50 messages from the MongoDB database, sorted by timestamp.
* These messages are sent only to the connecting client using `socket.emit`.

---

### 3⃣ Sending New Messages

```js
socket.on('send_message', async (data) => {
    const savedMessage = await saveMessage(data);
    io.emit('receive_message', savedMessage);
});
```

* When a user sends a new message, it triggers the `send_message` event.
* The server saves the message and immediately broadcasts it to **all** connected users using `io.emit`.

---

### 4⃣ Saving Messages to the Database

```js
const newMessage = new Message({
    user: data.user,
    text: data.text,
    fileUrl: data.fileUrl
});
return await newMessage.save();
```

* Each message (with user name, text, and optional image URL) is saved to MongoDB.
* This ensures that message history can be retrieved when a new client connects.

---

### 5⃣ Disconnecting the Client

```js
socket.on('disconnect', () => {
    console.log('Client disconnected');
});
```

* When the user closes the tab or loses connection, the `disconnect` event is triggered and logged.

---

## 🔄 Socket.io Event Flow (Summary)

```
User connects
   ↓
Server sends last 50 messages to the user
   ↓
User sends new message
   ↓
Server saves message to database
   ↓
Server broadcasts message to all clients
   ↓
All users see the message in real time
   ↓
User disconnects
```

---

## 🔁 socket.emit vs io.emit

| Method        | Sends To                    |
| ------------- | --------------------------- |
| `socket.emit` | Only the **current** client |
| `io.emit`     | **All** connected clients   |

---