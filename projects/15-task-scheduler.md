# Project 15: Task Scheduler API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md)

---

## 📝 Description

Build a task scheduling system with recurring tasks, reminders, and priority management.

**What You'll Learn:**
- Cron expressions
- Recurring tasks logic
- Priority queues
- Deadline management
- Notification scheduling

---

## 🎯 Project Goals

- ✅ Create one-time and recurring tasks
- ✅ Set priorities and deadlines
- ✅ Mark tasks complete
- ✅ Get tasks for specific dates
- ✅ React calendar view with tasks

---

## 🗺️ Roadmap

### Step 1: Task Model
**Todo:**
- [ ] Task structure:
  - title, description
  - dueDate, priority (low, medium, high)
  - status (pending, in-progress, completed)
  - recurrence (none, daily, weekly, monthly)
  - reminder (minutes before due)
  - assignedTo, createdBy

---

### Step 2: Task CRUD
**Todo:**
- [ ] `POST /api/tasks` - Create task
- [ ] `GET /api/tasks` - All tasks
- [ ] `GET /api/tasks/:id` - Single task
- [ ] `PATCH /api/tasks/:id/status` - Update status
- [ ] `DELETE /api/tasks/:id` - Delete

---

### Step 3: Recurring Tasks
**Todo:**
- [ ] Create recurring task template
- [ ] Generate instances based on recurrence
- [ ] `GET /api/tasks/date/:date` - Tasks for specific date
- [ ] When task completed, auto-create next instance

**Recurrence Logic:**
- Daily: Add 1 day to dueDate
- Weekly: Add 7 days
- Monthly: Add 1 month

---

### Step 4: Priority & Filtering
**Todo:**
- [ ] `GET /api/tasks?priority=high`
- [ ] `GET /api/tasks?status=pending`
- [ ] `GET /api/tasks?overdue=true`
- [ ] Sort by: priority, dueDate, status

**Overdue Logic:**
```
if (task.dueDate < currentDate && task.status !== 'completed') {
  task.overdue = true;
}
```

---

### Step 5: Reminders
**Todo:**
- [ ] Calculate reminder time (dueDate - reminder minutes)
- [ ] `GET /api/tasks/reminders/upcoming` - Tasks with reminders in next hour
- [ ] Could trigger email/notification (advanced)

---

### Step 6: Task Dependencies (Advanced)
**Todo:**
- [ ] Task A depends on Task B
- [ ] Can't start Task A until Task B is complete
- [ ] Show task tree/hierarchy

---

### Step 7: React Frontend
**Todo:**
- [ ] Task list with filters
- [ ] Calendar view
- [ ] Add task modal
- [ ] Priority badges (colors)
- [ ] Drag-and-drop to reschedule
- [ ] Mark complete checkbox

---

## 💡 Hints & Tips

### Checking Overdue:
```javascript
const now = new Date();
const overdue = tasks.filter(t => 
  new Date(t.dueDate) < now && 
  t.status !== 'completed'
);
```

### Next Recurrence:
```javascript
function getNextDate(currentDate, recurrence) {
  const next = new Date(currentDate);
  if (recurrence === 'daily') next.setDate(next.getDate() + 1);
  if (recurrence === 'weekly') next.setDate(next.getDate() + 7);
  if (recurrence === 'monthly') next.setMonth(next.getMonth() + 1);
  return next;
}
```

---

## 📚 Concepts

- Date manipulation
- Recurring logic
- Priority systems
- Status workflows
- Calendar integration

---

**Next:** [Project 16: Library Management System](16-library-management.md)
