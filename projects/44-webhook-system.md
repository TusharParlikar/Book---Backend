# Project 44: Webhook System

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Build a webhook system to notify external services about events in your application.

**What You'll Learn:**
- Webhook implementation
- Event-driven architecture
- Retry mechanisms
- Webhook security
- Payload signing

---

## 🎯 Project Goals

- ✅ Register webhook URLs
- ✅ Trigger webhooks on events
- ✅ Secure webhook delivery
- ✅ Retry failed webhooks
- ✅ React webhook management UI

---

## 🗺️ Roadmap

### Step 1: Webhook Model
**Todo:**
- [ ] Webhook structure:
  - url, events array (order.created, user.updated)
  - secret (for signature verification)
  - isActive, userId
  - createdAt

---

### Step 2: Register Webhook
**Todo:**
- [ ] `POST /api/webhooks`
  - Body: { url, events }
  - Generate secret for signature
  - Validate URL format
  - Test webhook with ping event
  - Save to database

---

### Step 3: Trigger Webhook
**Todo:**
- [ ] When event occurs (e.g., order created):
  ```javascript
  await triggerWebhook('order.created', orderData);
  ```
- [ ] Find all webhooks subscribed to this event
- [ ] Send POST request to each URL
- [ ] Include event and data in payload

---

### Step 4: Secure Delivery
**Todo:**
- [ ] Sign payload with HMAC
- [ ] Include signature in header: `X-Webhook-Signature`
- [ ] Receiver can verify signature

**Signature Generation:**
```javascript
const crypto = require('crypto');
const signature = crypto
  .createHmac('sha256', webhook.secret)
  .update(JSON.stringify(payload))
  .digest('hex');

headers['X-Webhook-Signature'] = signature;
```

---

### Step 5: Retry Mechanism
**Todo:**
- [ ] If webhook fails (network error, 5xx):
  - Retry after 1 min
  - Retry after 5 min
  - Retry after 15 min
  - Max 3 retries
- [ ] Log delivery attempts
- [ ] Disable webhook after repeated failures

---

### Step 6: Webhook Logs
**Todo:**
- [ ] Log every webhook delivery:
  - webhookId, event, payload
  - status (success/failed)
  - responseCode, responseBody
  - attemptNumber, timestamp
- [ ] `GET /api/webhooks/:id/logs` - View delivery history

---

### Step 7: Test Webhook
**Todo:**
- [ ] `POST /api/webhooks/:id/test`
- [ ] Send test ping event
- [ ] Return delivery result

---

### Step 8: React UI
**Todo:**
- [ ] Webhook list page
- [ ] Add webhook form
- [ ] Select events (checkboxes)
- [ ] View delivery logs
- [ ] Retry failed deliveries
- [ ] Enable/disable webhooks

---

## 💡 Hints & Tips

### Trigger Webhook:
```javascript
async function triggerWebhook(event, data) {
  const webhooks = await Webhook.find({ 
    events: event, 
    isActive: true 
  });
  
  for (const webhook of webhooks) {
    await deliverWebhook(webhook, event, data);
  }
}

async function deliverWebhook(webhook, event, data) {
  const payload = { event, data, timestamp: Date.now() };
  const signature = generateSignature(payload, webhook.secret);
  
  try {
    const response = await axios.post(webhook.url, payload, {
      headers: { 'X-Webhook-Signature': signature },
      timeout: 5000
    });
    
    logDelivery(webhook.id, 'success', response.status);
  } catch (error) {
    logDelivery(webhook.id, 'failed', error.message);
    scheduleRetry(webhook, payload);
  }
}
```

### Verify Signature (Receiver Side):
```javascript
function verifyWebhook(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return signature === expectedSignature;
}
```

---

## 📚 Concepts

- Webhook system
- Event-driven architecture
- HMAC signatures
- Retry mechanisms
- Async processing
- Security

---

## 🎨 Enhancement Ideas

- Rate limiting per webhook
- Webhook playground for testing
- Event filtering (e.g., only orders > $100)
- Webhook analytics
- Circuit breaker pattern

---

**Next:** [Project 45: API Rate Limiting with Redis](45-api-rate-limiting-redis.md)
