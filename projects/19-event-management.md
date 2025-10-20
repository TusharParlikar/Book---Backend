# Project 19: Event Management API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 4 hours  
**Related Chapter:** [B06 - Database Relationships](../chapters/B06_DATABASE_RELATIONSHIPS.md), [B08 - JWT Authentication](../chapters/B08_JWT_AUTHENTICATION.md)

---

## 📝 Description

Build an event booking platform with ticketing, capacity management, and attendee tracking.

**What You'll Learn:**
- Event management
- Ticket booking system
- Capacity constraints
- QR code generation
- Calendar integration

---

## 🎯 Project Goals

- ✅ Create and manage events
- ✅ Book tickets with capacity limits
- ✅ Generate ticket QR codes
- ✅ Check-in system
- ✅ React event booking UI

---

## 🗺️ Roadmap

### Step 1: Event Model
**Todo:**
- [ ] title, description, venue
- [ ] startDate, endDate, organizer
- [ ] capacity, bookedTickets
- [ ] price, category, image
- [ ] status (upcoming/ongoing/completed)

---

### Step 2: Booking System
**Todo:**
- [ ] `POST /api/bookings`
  - Body: { eventId, tickets }
  - Check capacity: bookedTickets + tickets ≤ capacity
  - Generate unique booking code
  - Decrease available seats
- [ ] `GET /api/bookings/my` - User's bookings
- [ ] `DELETE /api/bookings/:id` - Cancel (refund logic)

---

### Step 3: Ticket Generation
**Todo:**
- [ ] Generate QR code for each booking
- [ ] QR contains: bookingId, eventId, tickets
- [ ] Use `qrcode` package
- [ ] `GET /api/bookings/:id/ticket` - Download ticket

---

### Step 4: Check-In System
**Todo:**
- [ ] `POST /api/events/:id/checkin`
  - Body: { bookingCode }
  - Verify booking exists
  - Mark as checked-in
  - Prevent duplicate check-in

---

### Step 5: Event Discovery
**Todo:**
- [ ] `GET /api/events?category=Music`
- [ ] `GET /api/events?city=New York`
- [ ] `GET /api/events?date=2025-02-01`
- [ ] Sort by: date, popularity, price

---

### Step 6: Calendar Integration
**Todo:**
- [ ] `GET /api/events/calendar` - Return events in calendar format
- [ ] Generate .ics file for adding to Google Calendar

---

### Step 7: React Frontend
**Todo:**
- [ ] Event listings
- [ ] Event detail with booking
- [ ] Ticket purchase flow
- [ ] My tickets page
- [ ] QR code display
- [ ] Event calendar view

---

## 💡 Hints & Tips

### Capacity Check:
```javascript
if (event.bookedTickets + requestedTickets > event.capacity) {
  return res.status(400).json({ error: 'Not enough seats' });
}
```

### QR Code Generation:
```javascript
const QRCode = require('qrcode');
const qrData = JSON.stringify({ bookingId, eventId });
const qrImage = await QRCode.toDataURL(qrData);
```

---

## 📚 Concepts

- Booking systems
- Capacity management
- QR code generation
- Calendar integration
- Transaction safety

---

**Next:** [Project 20: Chat Messages API](20-chat-messages.md)
