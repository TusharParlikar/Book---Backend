# Project 51: SaaS Multi-Tenant Platform with AI & Payments 🚀

**Build a Complete SaaS Application - Job-Ready Full-Stack Project**

---

## 🎯 Project Overview

Build a **complete SaaS platform** like Notion, Slack, or Stripe - with multi-tenancy, AI features, payment subscriptions, and real-time collaboration.

**Difficulty:** Expert ⭐⭐⭐⭐⭐  
**Time Required:** 40-50 hours  
**Skills Used:** ALL chapters (B01-B20)

---

## 💡 What You'll Build

**Product:** AI-Powered Project Management SaaS

### Core Features:
1. **Multi-Tenant Architecture** - Each company gets isolated workspace
2. **Subscription Billing** - Razorpay integration with tiered plans
3. **AI Assistant** - OpenAI chatbot for task suggestions
4. **Real-Time Collaboration** - Live updates with Socket.io
5. **Role-Based Access Control** - Admin, Manager, Member roles
6. **File Uploads** - Cloudinary for documents and images
7. **Analytics Dashboard** - Real-time metrics with charts
8. **Email Notifications** - Automated workflows
9. **API Documentation** - Swagger/OpenAPI
10. **Dockerized Deployment** - Production-ready containers

---

## 🏗️ System Architecture

```
┌─────────────────┐
│   React SPA     │
│  (TypeScript)   │
└────────┬────────┘
         │
    ┌────▼────────────────────────────┐
    │   Node.js API Gateway           │
    │   - JWT Auth                    │
    │   - Rate Limiting               │
    │   - Request Validation          │
    └────┬────────────────────────────┘
         │
    ┌────┴──────────┬──────────────┬──────────────┐
    │               │              │              │
┌───▼────┐   ┌─────▼──────┐  ┌───▼───────┐  ┌──▼─────────┐
│MongoDB │   │Python ML    │  │Redis      │  │Socket.io   │
│Replica │   │Service      │  │Cache      │  │Server      │
│Set     │   │(AI/Analytics│  │           │  │            │
└────────┘   └────────────┘  └───────────┘  └────────────┘
```

---

## 📋 Step-by-Step Roadmap

### Phase 1: Foundation (10 hours)

#### Step 1.1: Database Design

**Workspace (Tenant) Model:**
```javascript
{
  _id: ObjectId,
  name: "Acme Corp",
  subdomain: "acme", // acme.yourapp.com
  plan: "pro", // free, basic, pro, enterprise
  subscriptionStatus: "active",
  razorpaySubscriptionId: String,
  features: {
    maxUsers: 50,
    maxProjects: 100,
    aiEnabled: true,
    customDomain: false
  },
  billing: {
    email: String,
    nextBillingDate: Date,
    amount: Number
  },
  createdAt: Date
}
```

**User Model (Multi-Tenant):**
```javascript
{
  _id: ObjectId,
  email: String, // Unique globally
  password: String,
  workspaces: [{
    workspace: ObjectId,
    role: "admin" | "manager" | "member",
    joinedAt: Date
  }],
  currentWorkspace: ObjectId, // Active workspace
  profile: {
    name: String,
    avatar: String,
    phone: String
  }
}
```

**Project Model:**
```javascript
{
  _id: ObjectId,
  workspace: ObjectId, // Tenant isolation
  name: String,
  description: String,
  status: "active" | "completed" | "archived",
  members: [{
    user: ObjectId,
    role: "owner" | "contributor" | "viewer"
  }],
  tasks: [{
    title: String,
    description: String,
    assignee: ObjectId,
    status: "todo" | "in-progress" | "done",
    priority: "low" | "medium" | "high",
    dueDate: Date,
    aiGenerated: Boolean,
    createdAt: Date
  }],
  files: [{
    name: String,
    url: String,
    uploadedBy: ObjectId,
    uploadedAt: Date
  }]
}
```

#### Step 1.2: Authentication & Multi-Tenancy

**Challenges:**
- User can belong to multiple workspaces
- Workspace data must be completely isolated
- Subdomain routing (acme.yourapp.com)

**Implementation:**
```javascript
// Middleware: Extract workspace from subdomain
const extractWorkspace = async (req, res, next) => {
  const subdomain = req.hostname.split('.')[0];
  
  if (subdomain === 'app' || subdomain === 'www') {
    return next(); // Main domain
  }
  
  const workspace = await Workspace.findOne({ subdomain });
  
  if (!workspace) {
    return res.status(404).json({ message: 'Workspace not found' });
  }
  
  req.workspace = workspace;
  next();
};

// Middleware: Verify user belongs to workspace
const verifyWorkspaceAccess = async (req, res, next) => {
  const user = req.user; // From JWT
  const workspaceId = req.workspace._id;
  
  const hasAccess = user.workspaces.some(
    w => w.workspace.toString() === workspaceId.toString()
  );
  
  if (!hasAccess) {
    return res.status(403).json({ message: 'Access denied to this workspace' });
  }
  
  next();
};

// Usage
app.use(extractWorkspace);
app.use('/api', auth, verifyWorkspaceAccess, routes);
```

#### Step 1.3: Subscription & Billing

**Create Razorpay Plans:**
```javascript
const PLANS = {
  free: {
    name: 'Free',
    price: 0,
    features: {
      maxUsers: 3,
      maxProjects: 5,
      aiEnabled: false
    }
  },
  pro: {
    name: 'Pro',
    price: 999, // ₹999/month
    razorpayPlanId: 'plan_xxxxx',
    features: {
      maxUsers: 50,
      maxProjects: 100,
      aiEnabled: true
    }
  },
  enterprise: {
    name: 'Enterprise',
    price: 4999,
    razorpayPlanId: 'plan_yyyyy',
    features: {
      maxUsers: 999,
      maxProjects: 999,
      aiEnabled: true,
      customDomain: true
    }
  }
};
```

**Subscribe Endpoint:**
```javascript
router.post('/billing/subscribe', auth, async (req, res) => {
  const { plan } = req.body; // 'pro' or 'enterprise'
  const workspace = await Workspace.findById(req.user.currentWorkspace);
  
  if (!PLANS[plan]) {
    return res.status(400).json({ message: 'Invalid plan' });
  }
  
  // Create Razorpay subscription
  const subscription = await razorpay.subscriptions.create({
    plan_id: PLANS[plan].razorpayPlanId,
    customer_notify: 1,
    total_count: 12, // 12 monthly payments
    notes: {
      workspaceId: workspace._id.toString()
    }
  });
  
  workspace.plan = plan;
  workspace.subscriptionStatus = 'pending';
  workspace.razorpaySubscriptionId = subscription.id;
  workspace.features = PLANS[plan].features;
  await workspace.save();
  
  res.json({
    success: true,
    subscriptionId: subscription.id,
    shortUrl: subscription.short_url
  });
});
```

**Webhook Handler:**
```javascript
router.post('/webhooks/razorpay', validateWebhook, async (req, res) => {
  const { event, payload } = req.body;
  
  if (event === 'subscription.activated') {
    const subscriptionId = payload.subscription.entity.id;
    
    await Workspace.updateOne(
      { razorpaySubscriptionId: subscriptionId },
      { subscriptionStatus: 'active' }
    );
  }
  
  if (event === 'subscription.cancelled') {
    const subscriptionId = payload.subscription.entity.id;
    
    const workspace = await Workspace.findOne({ razorpaySubscriptionId: subscriptionId });
    workspace.subscriptionStatus = 'cancelled';
    workspace.plan = 'free';
    workspace.features = PLANS.free.features;
    await workspace.save();
  }
  
  res.json({ status: 'ok' });
});
```

---

### Phase 2: AI Integration (8 hours)

#### Step 2.1: AI Task Assistant

**Feature:** User types "Create tasks for building a landing page" → AI generates subtasks

```javascript
router.post('/projects/:projectId/ai/generate-tasks', auth, async (req, res) => {
  const { prompt } = req.body;
  const workspace = await Workspace.findById(req.user.currentWorkspace);
  
  // Check if AI enabled for workspace
  if (!workspace.features.aiEnabled) {
    return res.status(403).json({ 
      message: 'Upgrade to Pro plan to use AI features' 
    });
  }
  
  // Call OpenAI
  const completion = await openai.chat.completions.create({
    model: 'gpt-3.5-turbo',
    messages: [
      {
        role: 'system',
        content: `You are a project management assistant. Generate 5-10 specific, actionable tasks based on the user's request. Return as JSON array with fields: title, description, priority (low/medium/high), estimatedHours.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    response_format: { type: 'json_object' }
  });
  
  const aiTasks = JSON.parse(completion.choices[0].message.content);
  
  // Add to project
  const project = await Project.findById(req.params.projectId);
  
  aiTasks.tasks.forEach(task => {
    project.tasks.push({
      title: task.title,
      description: task.description,
      priority: task.priority,
      status: 'todo',
      aiGenerated: true,
      createdAt: new Date()
    });
  });
  
  await project.save();
  
  // Track AI usage
  await AIUsage.create({
    workspace: workspace._id,
    user: req.user._id,
    action: 'generate-tasks',
    tokensUsed: completion.usage.total_tokens,
    cost: (completion.usage.total_tokens / 1000) * 0.002
  });
  
  res.json({
    success: true,
    tasksGenerated: aiTasks.tasks.length,
    tasks: project.tasks
  });
});
```

#### Step 2.2: AI Chatbot for Support

```javascript
router.post('/ai/chat', auth, async (req, res) => {
  const { message, conversationId } = req.body;
  
  // Fetch conversation history
  let conversation = await Conversation.findById(conversationId);
  
  if (!conversation) {
    conversation = await Conversation.create({
      user: req.user._id,
      workspace: req.user.currentWorkspace,
      messages: []
    });
  }
  
  conversation.messages.push({
    role: 'user',
    content: message
  });
  
  // Build context with workspace data
  const recentProjects = await Project.find({
    workspace: req.user.currentWorkspace
  }).limit(5).select('name status tasks');
  
  const contextMessage = `
Context: User is from workspace with ${recentProjects.length} active projects.
Recent projects: ${recentProjects.map(p => p.name).join(', ')}

Answer user questions about their projects, tasks, and provide productivity tips.
  `;
  
  // Call OpenAI with function calling
  const completion = await openai.chat.completions.create({
    model: 'gpt-3.5-turbo',
    messages: [
      { role: 'system', content: contextMessage },
      ...conversation.messages.map(m => ({
        role: m.role,
        content: m.content
      }))
    ],
    functions: [
      {
        name: 'get_project_stats',
        description: 'Get statistics for a specific project',
        parameters: {
          type: 'object',
          properties: {
            projectId: { type: 'string' }
          }
        }
      }
    ]
  });
  
  const aiReply = completion.choices[0].message.content;
  
  conversation.messages.push({
    role: 'assistant',
    content: aiReply
  });
  
  await conversation.save();
  
  res.json({
    success: true,
    reply: aiReply,
    conversationId: conversation._id
  });
});
```

---

### Phase 3: Real-Time Features (8 hours)

#### Step 3.1: Live Task Updates

```javascript
// Socket.io with workspace rooms
io.on('connection', (socket) => {
  const user = socket.user; // From auth middleware
  const workspaceId = user.currentWorkspace.toString();
  
  // Join workspace room
  socket.join(`workspace:${workspaceId}`);
  
  // User updates task
  socket.on('task:update', async (data) => {
    const { projectId, taskId, updates } = data;
    
    const project = await Project.findById(projectId);
    const task = project.tasks.id(taskId);
    
    Object.assign(task, updates);
    await project.save();
    
    // Broadcast to all workspace members
    io.to(`workspace:${workspaceId}`).emit('task:updated', {
      projectId,
      taskId,
      updates,
      updatedBy: user.profile.name
    });
  });
  
  // User is viewing a project
  socket.on('project:join', (projectId) => {
    socket.join(`project:${projectId}`);
    
    // Show who's viewing
    io.to(`project:${projectId}`).emit('user:viewing', {
      userId: user._id,
      name: user.profile.name
    });
  });
});
```

#### Step 3.2: Real-Time Notifications

```javascript
// When task assigned
router.post('/projects/:projectId/tasks/:taskId/assign', auth, async (req, res) => {
  const { assigneeId } = req.body;
  
  const project = await Project.findById(req.params.projectId);
  const task = project.tasks.id(req.params.taskId);
  
  task.assignee = assigneeId;
  await project.save();
  
  // Send real-time notification
  io.to(`user:${assigneeId}`).emit('notification', {
    type: 'task-assigned',
    title: 'New Task Assigned',
    message: `You've been assigned: ${task.title}`,
    data: {
      projectId: project._id,
      taskId: task._id
    }
  });
  
  // Send email
  const assignee = await User.findById(assigneeId);
  await sendEmail(assignee.email, 'Task Assigned', `...`);
  
  res.json({ success: true, task });
});
```

---

### Phase 4: Analytics & Monitoring (6 hours)

#### Step 4.1: Real-Time Workspace Analytics

```javascript
router.get('/analytics/workspace', auth, async (req, res) => {
  const workspaceId = req.user.currentWorkspace;
  
  const [projects, tasks, members, activity] = await Promise.all([
    Project.countDocuments({ workspace: workspaceId }),
    Project.aggregate([
      { $match: { workspace: workspaceId } },
      { $unwind: '$tasks' },
      { $group: {
        _id: '$tasks.status',
        count: { $sum: 1 }
      }}
    ]),
    User.countDocuments({
      'workspaces.workspace': workspaceId
    }),
    ActivityLog.find({ workspace: workspaceId })
      .sort({ createdAt: -1 })
      .limit(20)
  ]);
  
  // Task completion rate
  const totalTasks = tasks.reduce((sum, t) => sum + t.count, 0);
  const completedTasks = tasks.find(t => t._id === 'done')?.count || 0;
  const completionRate = (completedTasks / totalTasks * 100).toFixed(2);
  
  res.json({
    projects,
    members,
    tasks: {
      total: totalTasks,
      byStatus: tasks,
      completionRate
    },
    recentActivity: activity
  });
});
```

#### Step 4.2: Python Analytics Service

Generate weekly reports with charts using Pandas and Matplotlib.

---

### Phase 5: React Frontend (10 hours)

#### Step 5.1: Workspace Switcher

```jsx
function WorkspaceSwitcher() {
  const { user, setCurrentWorkspace } = useAuth();
  
  return (
    <Dropdown>
      {user.workspaces.map(ws => (
        <DropdownItem
          key={ws.workspace._id}
          onClick={() => setCurrentWorkspace(ws.workspace._id)}
        >
          {ws.workspace.name}
          <Badge>{ws.role}</Badge>
        </DropdownItem>
      ))}
      <DropdownDivider />
      <DropdownItem onClick={() => setShowCreateWorkspace(true)}>
        + Create Workspace
      </DropdownItem>
    </Dropdown>
  );
}
```

#### Step 5.2: Real-Time Dashboard

```jsx
function ProjectBoard() {
  const [project, setProject] = useState(null);
  const [socket, setSocket] = useState(null);
  
  useEffect(() => {
    const newSocket = io('http://localhost:5000', {
      auth: { token: localStorage.getItem('token') }
    });
    
    newSocket.emit('project:join', projectId);
    
    newSocket.on('task:updated', ({ taskId, updates, updatedBy }) => {
      setProject(prev => ({
        ...prev,
        tasks: prev.tasks.map(t =>
          t._id === taskId ? { ...t, ...updates } : t
        )
      }));
      
      toast.info(`${updatedBy} updated a task`);
    });
    
    setSocket(newSocket);
    
    return () => newSocket.close();
  }, [projectId]);
  
  const handleTaskDragEnd = async (result) => {
    // Update locally
    const newTasks = reorderTasks(project.tasks, result);
    setProject({ ...project, tasks: newTasks });
    
    // Emit to others via Socket.io
    socket.emit('task:update', {
      projectId,
      taskId: result.draggableId,
      updates: { status: result.destination.droppableId }
    });
  };
  
  return (
    <DragDropContext onDragEnd={handleTaskDragEnd}>
      <Board>
        <Column status="todo" tasks={todoTasks} />
        <Column status="in-progress" tasks={inProgressTasks} />
        <Column status="done" tasks={doneTasks} />
      </Board>
    </DragDropContext>
  );
}
```

---

## 🎨 UI/UX Features

1. **Dashboard**: Revenue, active projects, task completion rate
2. **Kanban Board**: Drag-and-drop tasks (like Trello)
3. **Gantt Chart**: Timeline view
4. **AI Chatbot**: Floating chat widget
5. **Notifications**: Real-time toast notifications
6. **File Preview**: PDF, images, docs
7. **Dark Mode**: Theme switcher
8. **Keyboard Shortcuts**: Power user features

---

## 🚀 Deployment

### Docker Setup

**File:** `docker-compose.yml`

```yaml
version: '3.8'

services:
  api:
    build: ./backend
    ports:
      - "5000:5000"
    env_file: .env
    depends_on:
      - mongo
      - redis
      
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
      
  mongo:
    image: mongo:6
    volumes:
      - mongo-data:/data/db
      
  redis:
    image: redis:alpine
    
  ml-service:
    build: ./ml-service
    ports:
      - "5001:5001"

volumes:
  mongo-data:
```

---

## ✅ Deliverables

- [ ] Multi-tenant authentication system
- [ ] Razorpay subscription billing
- [ ] AI task generation with OpenAI
- [ ] Real-time collaboration with Socket.io
- [ ] Role-based access control
- [ ] File uploads to Cloudinary
- [ ] Analytics dashboard with charts
- [ ] Email notifications
- [ ] Swagger API documentation
- [ ] Docker deployment
- [ ] React TypeScript frontend
- [ ] 90%+ test coverage

---

## 📚 Technologies Used

**Backend:** Node.js, Express, MongoDB, Redis, Socket.io  
**Frontend:** React, TypeScript, TailwindCSS, Zustand  
**AI:** OpenAI GPT-3.5/4  
**Payments:** Razorpay  
**DevOps:** Docker, GitHub Actions  
**Testing:** Jest, Supertest, React Testing Library  

---

## 🎓 Learning Outcomes

After completing this project, you will have:

✅ Built a production-ready SaaS application  
✅ Implemented multi-tenancy architecture  
✅ Integrated AI and payment systems  
✅ Mastered real-time features  
✅ Created a portfolio-worthy project  
✅ **Ready for senior developer roles!**

---

**Related Chapters:** B01-B20 (All chapters)

**Made with ❤️ by Tushar Parlikar**

[← Back to Projects](README.md) | [Next Project: AI E-Commerce Platform →](52-ai-ecommerce-platform.md)
