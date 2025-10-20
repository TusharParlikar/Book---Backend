# Project 16: Library Management System

**Difficulty:** 🟡 Medium  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md), [B06 - Database Relationships](../chapters/B06_DATABASE_RELATIONSHIPS.md)

---

## 📝 Description

Build a complete library management system with books, members, borrowing, and fines.

**What You'll Learn:**
- Database relationships (books ↔ members)
- Transaction management (borrow/return)
- Date calculations (due dates, fines)
- Inventory management
- Business logic implementation

---

## 🎯 Project Goals

- ✅ Manage books and members
- ✅ Borrow and return books
- ✅ Calculate fines for late returns
- ✅ Track book availability
- ✅ React library dashboard

---

## 🗺️ Roadmap

### Step 1: Models

**Books:**
- [ ] id, title, author, ISBN
- [ ] genre, publishYear, copies
- [ ] availableCopies, totalCopies
- [ ] description, cover image

**Members:**
- [ ] id, name, email, phone
- [ ] membershipNumber
- [ ] joinedDate, status (active/inactive)
- [ ] borrowedBooks (array of book IDs)

**Transactions:**
- [ ] id, bookId, memberId
- [ ] borrowDate, dueDate, returnDate
- [ ] fine, status (borrowed/returned/overdue)

---

### Step 2: Books Management
**Todo:**
- [ ] `POST /api/books` - Add book
- [ ] `GET /api/books` - List all
- [ ] `GET /api/books/available` - Only available books
- [ ] `PUT /api/books/:id` - Update
- [ ] `DELETE /api/books/:id` - Remove

---

### Step 3: Members Management
**Todo:**
- [ ] `POST /api/members` - Register member
- [ ] `GET /api/members` - All members
- [ ] `GET /api/members/:id` - Member details + borrowed books
- [ ] `PUT /api/members/:id` - Update info
- [ ] `PATCH /api/members/:id/status` - Activate/deactivate

---

### Step 4: Borrowing System
**Todo:**
- [ ] `POST /api/transactions/borrow`
  - Body: { bookId, memberId }
  - Check: book available, member active
  - Decrease availableCopies
  - Create transaction record
  - Set dueDate (14 days from now)
- [ ] Return error if book unavailable

---

### Step 5: Return System
**Todo:**
- [ ] `POST /api/transactions/return`
  - Body: { transactionId }
  - Set returnDate to today
  - Increase availableCopies
  - Calculate fine if overdue
  - Update transaction status

**Fine Calculation:**
```
if (returnDate > dueDate) {
  daysLate = difference in days
  fine = daysLate * finePerDay ($1/day)
}
```

---

### Step 6: Reports & Analytics
**Todo:**
- [ ] `GET /api/reports/most-borrowed` - Popular books
- [ ] `GET /api/reports/overdue` - All overdue books
- [ ] `GET /api/reports/member/:id/history` - Borrowing history
- [ ] `GET /api/reports/revenue` - Total fines collected

---

### Step 7: React Frontend
**Todo:**
- [ ] Books catalog with search
- [ ] Member registration form
- [ ] Borrow book interface
- [ ] Return book interface
- [ ] Overdue books list
- [ ] Member dashboard showing borrowed books

---

## 💡 Hints & Tips

### Check Book Availability:
```javascript
if (book.availableCopies > 0) {
  // Allow borrowing
}
```

### Due Date (14 days):
```javascript
const dueDate = new Date();
dueDate.setDate(dueDate.getDate() + 14);
```

### Calculate Days Late:
```javascript
const diffTime = returnDate - dueDate;
const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
```

### Update Copies on Borrow:
```javascript
book.availableCopies -= 1;
```

---

## 📚 Concepts

- Database relationships
- Transaction management
- Date calculations
- Inventory tracking
- Fine calculations
- Business logic

---

**Next:** [Project 17: Course Management System](17-course-management.md)
