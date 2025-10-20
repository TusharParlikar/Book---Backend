# Chapter 20: Real-Time Analytics & WebSockets 🔴

**Build Live Dashboards with Socket.io and Event-Driven Architecture**

---

## 📚 What You'll Learn

- Real-time data streaming with WebSockets
- Socket.io for live updates
- Building live analytics dashboards
- Server-Sent Events (SSE)
- Redis for pub/sub messaging
- Real-time notifications
- Live monitoring and alerting
- Scaling WebSocket connections

**Time Required:** 8-10 hours  
**Difficulty:** Expert  
**Prerequisites:** B01-B19

---

## 🎯 Learning Objectives

By the end of this chapter, you will:

✅ Implement real-time communication with Socket.io  
✅ Stream live analytics to dashboards  
✅ Build event-driven architectures  
✅ Use Redis for pub/sub messaging  
✅ Create live notifications system  
✅ Monitor application metrics in real-time  
✅ Handle millions of WebSocket connections  
✅ Build production-ready real-time apps  

---

## 🚀 Socket.io Setup

### Install Dependencies

```bash
npm install socket.io socket.io-client redis ioredis
```

<details>
<summary>💡 Expected Output</summary>

```bash
added 4 packages, and audited 5 packages in 2s
found 0 vulnerabilities
```
</details>

### Basic Server Setup

**File:** `server.js`

```javascript
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: {
    origin: process.env.CLIENT_URL || 'http://localhost:3000',
    credentials: true
  }
});

// Socket.io connection handler
io.on('connection', (socket) => {
  console.log(`User connected: ${socket.id}`);

  socket.on('disconnect', () => {
    console.log(`User disconnected: ${socket.id}`);
  });
});

httpServer.listen(5000, () => {
  console.log('Server running on port 5000');
});

module.exports = { app, io };
```

---

## 📊 Live Analytics Dashboard

### Real-Time Order Tracking

```javascript
const Order = require('./models/Order');

// When new order is created
router.post('/orders', auth, async (req, res) => {
  const order = await Order.create({
    user: req.user._id,
    items: req.body.items,
    totalAmount: req.body.totalAmount
  });

  // Emit to all connected admin dashboards
  io.to('admin-room').emit('new-order', {
    orderId: order._id,
    totalAmount: order.totalAmount,
    timestamp: new Date()
  });

  // Update real-time metrics
  const todaySales = await Order.aggregate([
    { $match: { createdAt: { $gte: new Date(new Date().setHours(0, 0, 0, 0)) } } },
    { $group: { _id: null, totalRevenue: { $sum: '$totalAmount' }, orderCount: { $sum: 1 } } }
  ]);

  io.to('admin-room').emit('sales-update', todaySales[0] || { totalRevenue: 0, orderCount: 0 });

  res.status(201).json({ success: true, order });
});
```

### Admin Dashboard Connection

```javascript
// Middleware to authenticate socket connections
const authenticateSocket = async (socket, next) => {
  const token = socket.handshake.auth.token;
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findById(decoded.id);
    
    if (!user) {
      return next(new Error('Authentication error'));
    }
    
    socket.user = user;
    next();
  } catch (error) {
    next(new Error('Authentication error'));
  }
};

io.use(authenticateSocket);

io.on('connection', async (socket) => {
  const user = socket.user;

  // Join admin room if user is admin
  if (user.role === 'admin') {
    socket.join('admin-room');
    console.log(`Admin ${user.email} joined dashboard`);

    // Send initial data
    const realTimeData = await getRealTimeMetrics();
    socket.emit('initial-data', realTimeData);
  }

  // Join user's personal room for notifications
  socket.join(`user-${user._id}`);

  socket.on('disconnect', () => {
    console.log(`User ${user.email} disconnected`);
  });
});
```

### Real-Time Metrics Function

```javascript
async function getRealTimeMetrics() {
  try {
    const today = new Date();
    today.setHours(0, 0, 0, 0);

    const fifteenMinutesAgo = new Date(Date.now() - 15 * 60 * 1000);

    const [orders, revenueResult, activeUsers, newUsers] = await Promise.all([
      Order.countDocuments({ createdAt: { $gte: today } }),
      Order.aggregate([
        { $match: { createdAt: { $gte: today }, status: 'paid' } },
        { $group: { _id: null, total: { $sum: '$totalAmount' } } }
      ]),
      Session.countDocuments({ lastActive: { $gte: fifteenMinutesAgo } }),
      User.countDocuments({ createdAt: { $gte: today } })
    ]);

    return {
      todayOrders: orders,
      todayRevenue: revenueResult[0]?.total || 0,
      activeUsers,
      newUsers
    };
  } catch (error) {
    console.error("Error fetching real-time metrics:", error);
    return { todayOrders: 0, todayRevenue: 0, activeUsers: 0, newUsers: 0 };
  }
}

// Update metrics every 10 seconds
setInterval(async () => {
  const metrics = await getRealTimeMetrics();
  io.to('admin-room').emit('metrics-update', metrics);
}, 10000);
```

---

## 🔔 Real-Time Notifications

### Notification System

```javascript
const notificationSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  type: {
    type: String,
    enum: ['order', 'payment', 'message', 'alert'],
    required: true
  },
  title: String,
  message: String,
  data: mongoose.Schema.Types.Mixed,
  read: {
    type: Boolean,
    default: false
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

const Notification = mongoose.model('Notification', notificationSchema);
```

### Send Notification

```javascript
async function sendNotification(userId, type, title, message, data = {}) {
  // Save to database
  const notification = await Notification.create({
    user: userId,
    type,
    title,
    message,
    data
  });

  // Send real-time via Socket.io
  io.to(`user-${userId}`).emit('notification', {
    id: notification._id,
    type,
    title,
    message,
    data,
    timestamp: notification.createdAt
  });

  return notification;
}

// Example: Order confirmation
router.post('/orders/:id/confirm', auth, isAdmin, async (req, res) => {
  const order = await Order.findById(req.params.id);

  order.status = 'confirmed';
  await order.save();

  // Send notification to customer
  await sendNotification(
    order.user,
    'order',
    'Order Confirmed',
    `Your order #${order._id} has been confirmed and will be shipped soon.`,
    { orderId: order._id }
  );

  res.json({ success: true, order });
});
```

---

## 📡 Server-Sent Events (SSE)

Alternative to WebSockets for one-way server-to-client streaming.

### SSE Endpoint

```javascript
router.get('/stream/orders', auth, isAdmin, (req, res) => {
  // Set headers for SSE
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // Send initial comment
  res.write(': connected\n\n');

  // Listen for new orders
  const sendOrder = (order) => {
    res.write(`data: ${JSON.stringify(order)}\n\n`);
  };

  // MongoDB Change Streams
  const changeStream = Order.watch([
    {
      $match: {
        operationType: 'insert'
      }
    }
  ]);

  changeStream.on('change', (change) => {
    sendOrder(change.fullDocument);
  });

  // Cleanup on client disconnect
  req.on('close', () => {
    changeStream.close();
    res.end();
  });
});
```

### React SSE Client

```jsx
useEffect(() => {
  const eventSource = new EventSource('/api/stream/orders', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  eventSource.onmessage = (event) => {
    const order = JSON.parse(event.data);
    setOrders(prev => [order, ...prev]);
  };

  eventSource.onerror = () => {
    eventSource.close();
  };

  return () => eventSource.close();
}, []);
```

---

## 🔴 Redis Pub/Sub for Scaling

### Why Redis?

When you have multiple server instances, Socket.io needs Redis to sync events across servers.

```
User 1 → Server A → Redis Pub/Sub → Server B → User 2
```

### Setup Redis Adapter

```javascript
const { createAdapter } = require('@socket.io/redis-adapter');
const { createClient } = require('redis');

const pubClient = createClient({ url: process.env.REDIS_URL });
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  io.adapter(createAdapter(pubClient, subClient));
  console.log('Socket.io Redis adapter connected');
});
```

### Publish Events Across Servers

```javascript
const redis = require('./utils/redis');

// Publish order event
router.post('/orders', auth, async (req, res) => {
  const order = await Order.create(req.body);

  // Publish to Redis
  await redis.publish('orders', JSON.stringify({
    type: 'new-order',
    data: order
  }));

  res.json({ success: true, order });
});

// Subscribe to order events
redis.subscribe('orders');

redis.on('message', (channel, message) => {
  const event = JSON.parse(message);

  if (event.type === 'new-order') {
    io.to('admin-room').emit('new-order', event.data);
  }
});
```

---

## 📈 Live Monitoring Dashboard

### System Metrics

```javascript
const os = require('os');
const pidusage = require('pidusage');

// Emit system metrics every 5 seconds
setInterval(async () => {
  try {
    const stats = await pidusage(process.pid);
    const metrics = {
      cpu: stats.cpu.toFixed(2),
      memory: (stats.memory / 1024 / 1024).toFixed(2), // MB
      totalMemory: (os.totalmem() / 1024 / 1024 / 1024).toFixed(2), // GB
      freeMemory: (os.freemem() / 1024 / 1024 / 1024).toFixed(2), // GB
      uptime: process.uptime(),
      connections: io.engine.clientsCount
    };
    io.to('admin-room').emit('system-metrics', metrics);
  } catch (error) {
    console.error('Failed to get pidusage:', error);
  }
}, 5000);
```

### Database Metrics

```javascript
setInterval(async () => {
  const dbStats = await mongoose.connection.db.stats();

  const metrics = {
    collections: dbStats.collections,
    dataSize: (dbStats.dataSize / 1024 / 1024).toFixed(2), // MB
    storageSize: (dbStats.storageSize / 1024 / 1024).toFixed(2),
    indexes: dbStats.indexes,
    avgObjSize: dbStats.avgObjSize
  };

  io.to('admin-room').emit('db-metrics', metrics);
}, 10000);
```

---

## 🎨 React Live Dashboard

```jsx
import { useEffect, useState } from 'react';
import io from 'socket.io-client';

function LiveDashboard() {
  const [socket, setSocket] = useState(null);
  const [metrics, setMetrics] = useState(null);
  const [recentOrders, setRecentOrders] = useState([]);
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
  const newSocket = io('http://localhost:5000', {
    auth: { token: localStorage.getItem('token') },
    reconnectionAttempts: 5,
    reconnectionDelay: 1000,
  });

  setSocket(newSocket);

  newSocket.on('connect', () => console.log('Connected to server'));
  newSocket.on('disconnect', () => console.log('Disconnected from server'));

  newSocket.on('initial-data', setMetrics);
  newSocket.on('metrics-update', setMetrics);

  newSocket.on('new-order', (order) => {
    setRecentOrders(prev => [order, ...prev].slice(0, 10));
    toast.success(`New order: ₹${order.totalAmount}`);
  });

  newSocket.on('sales-update', (data) => {
    setMetrics(prev => ({
      ...prev,
      todayRevenue: data.totalRevenue,
      todayOrders: data.orderCount
    }));
  });

  return () => newSocket.close();
}, []);

  if (!metrics) return <div>Loading...</div>;

  return (
    <div className="live-dashboard">
      <h1>Live Analytics Dashboard 🔴</h1>

      {/* Metrics Cards */}
      <div className="metrics-grid">
        <div className="metric-card">
          <h3>Today's Revenue</h3>
          <p className="metric-value live">
            ₹{metrics.todayRevenue?.toLocaleString()}
          </p>
        </div>

        <div className="metric-card">
          <h3>Today's Orders</h3>
          <p className="metric-value live">{metrics.todayOrders}</p>
        </div>

        <div className="metric-card">
          <h3>Active Users</h3>
          <p className="metric-value live">{metrics.activeUsers}</p>
        </div>

        <div className="metric-card">
          <h3>New Users</h3>
          <p className="metric-value">{metrics.newUsers}</p>
        </div>
      </div>

      {/* Recent Orders */}
      <div className="recent-orders">
        <h2>Recent Orders (Live)</h2>
        <div className="orders-list">
          {recentOrders.map(order => (
            <div key={order.orderId} className="order-item animate-slide-in">
              <span className="order-id">#{order.orderId.slice(-6)}</span>
              <span className="order-amount">₹{order.totalAmount}</span>
              <span className="order-time">
                {new Date(order.timestamp).toLocaleTimeString()}
              </span>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

---

## 🔥 Real-Time Chat

### Chat Room Implementation

```javascript
io.on('connection', (socket) => {
  // Join chat room
  socket.on('join-room', (roomId) => {
    socket.join(roomId);
    console.log(`${socket.user.email} joined room ${roomId}`);

    // Notify others
    socket.to(roomId).emit('user-joined', {
      userId: socket.user._id,
      username: socket.user.name
    });
  });

  // Send message
  socket.on('send-message', async (data) => {
    const { roomId, message } = data;

    // Save message to database
    const chatMessage = await Message.create({
      room: roomId,
      user: socket.user._id,
      content: message,
      timestamp: new Date()
    });

    // Broadcast to room
    io.to(roomId).emit('new-message', {
      id: chatMessage._id,
      userId: socket.user._id,
      username: socket.user.name,
      content: message,
      timestamp: chatMessage.timestamp
    });
  });

  // Typing indicator
  socket.on('typing', (roomId) => {
    socket.to(roomId).emit('user-typing', {
      userId: socket.user._id,
      username: socket.user.name
    });
  });

  // Leave room
  socket.on('leave-room', (roomId) => {
    socket.leave(roomId);
    socket.to(roomId).emit('user-left', {
      userId: socket.user._id,
      username: socket.user.name
    });
  });
});
```

---

## 🎓 Hands-On Exercise

**Build: Live E-Commerce Monitoring System**

1. **Real-Time Analytics**:
   - Live sales counter
   - Order heatmap by region
   - Product views in real-time
   - Revenue ticker

2. **Live Notifications**:
   - New order alerts for admin
   - Order status updates for customers
   - Low stock alerts
   - Payment confirmations

3. **Live Chat Support**:
   - Customer-to-admin chat
   - Typing indicators
   - Read receipts
   - File sharing

4. **System Monitoring**:
   - CPU and memory usage
   - Active connections
   - Database performance
   - Error tracking

5. **Features**:
   - Auto-reconnect on disconnect
   - Offline message queue
   - Multiple rooms/channels
   - User presence tracking

---

## 📚 Practice Projects

- **Project #37:** [Real-Time Chat Application](../projects/37-realtime-chat.md) ⭐
- **Project #40:** [Analytics Dashboard](../projects/40-analytics-dashboard.md) ⭐
- **Project #53:** [Live Collaboration Tool](../projects/53-live-collaboration.md) ⭐

---

## 🔗 Resources

- [Socket.io Documentation](https://socket.io/docs/)
- [Redis Pub/Sub](https://redis.io/topics/pubsub)
- [Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

---

## ✅ Chapter Checklist

- [ ] Set up Socket.io server
- [ ] Implement authentication for sockets
- [ ] Build real-time analytics dashboard
- [ ] Create notification system
- [ ] Implement SSE for live updates
- [ ] Configure Redis adapter for scaling
- [ ] Build live chat functionality
- [ ] Monitor system metrics in real-time
- [ ] Handle reconnections gracefully
- [ ] Deploy with multiple server instances

---

## 🎯 What's Next?

**Congratulations!** 🎉 You've completed all 20 chapters! Now it's time to build job-ready full-stack projects that combine everything you've learned.

**Next:** Build 5 production-grade full-stack applications using all concepts, algorithms, and technologies from this course!

---

**Made with ❤️ by Tushar Parlikar**

[← Previous: Data Analysis with Python](B19_DATA_ANALYSIS.md) | [Projects Section →](../projects/README.md)
