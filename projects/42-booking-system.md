# Project 42: Booking System API

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 6-7 hours  
**Related Chapter:** [B10 - E-Commerce Backend](../chapters/B10_ECOMMERCE_BACKEND.md)

---

## 📝 Description

Build a reservation system for appointments, hotels, or services with availability management.

**What You'll Learn:**
- Time slot management
- Booking conflicts prevention
- Calendar availability
- Cancellation policies
- Reminder notifications

---

## 🎯 Project Goals

- ✅ Check availability
- ✅ Create bookings
- ✅ Prevent double-booking
- ✅ Cancellation handling
- ✅ React booking calendar UI

---

## 🗺️ Roadmap

### Step 1: Availability Model
**Todo:**
- [ ] Define available time slots
- [ ] resourceId (room, doctor, etc.)
- [ ] date, startTime, endTime
- [ ] isBooked, maxCapacity

### Step 2: Check Availability
**Todo:**
- [ ] `GET /api/availability?date=2025-02-01&resource=room1`
- [ ] Return free time slots
- [ ] Show booked vs available

### Step 3: Create Booking
**Todo:**
- [ ] `POST /api/bookings`
- [ ] Check: slot available
- [ ] Check: no conflicts
- [ ] Mark slot as booked
- [ ] Send confirmation email

### Step 4: Conflict Prevention
**Todo:**
- [ ] Use database transactions
- [ ] Lock time slot during booking
- [ ] Validate no overlapping bookings

**Algorithm:**
```
Check if any booking exists where:
  resource = requested resource AND
  (requested_start < existing_end AND requested_end > existing_start)
If exists: conflict!
```

### Step 5: Cancellation
**Todo:**
- [ ] `DELETE /api/bookings/:id`
- [ ] Free up the time slot
- [ ] Apply cancellation policy
- [ ] Refund logic

### Step 6: React Calendar
**Todo:**
- [ ] Month/week/day views
- [ ] Click to book
- [ ] Show availability
- [ ] Booking form

---

**Next:** [Project 43: Leaderboard System](43-leaderboard-system.md)
