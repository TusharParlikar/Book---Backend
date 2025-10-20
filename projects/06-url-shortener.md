# Project 6: URL Shortener

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Build a URL shortening service like bit.ly. Learn about hashing algorithms, redirects, and tracking analytics.

**What You'll Learn:**
- Hash generation algorithms
- HTTP redirects (301, 302)
- Base62 encoding
- Click tracking
- Custom short codes

---

## 🎯 Project Goals

- ✅ Shorten long URLs to 6-character codes
- ✅ Redirect short URLs to original URLs
- ✅ Track click count and analytics
- ✅ Custom short codes (optional)
- ✅ React UI to create and manage short URLs

---

## 🗺️ Roadmap

### Step 1: Understanding URL Shortening
**Todo:**
- [ ] Research how short codes are generated
- [ ] Understand Base62 encoding (a-z, A-Z, 0-9 = 62 characters)
- [ ] Plan data structure
- [ ] Calculate: 62^6 = 56 billion possible codes

**Data Structure:**
- originalUrl (full URL)
- shortCode (6-character code)
- customAlias (optional custom code)
- clicks (number)
- createdAt
- lastAccessedAt

---

### Step 2: Create Short Code Generator
**Todo:**
- [ ] Create `utils/codeGenerator.js`
- [ ] Implement Base62 encoding
- [ ] Write function to generate 6-character code
- [ ] Ensure codes are unique (check against database)
- [ ] Handle collisions (if code exists, regenerate)

**Algorithm Hint:**
```
Method 1: Random generation
- Generate random 6 characters from Base62 alphabet
- Check if exists, if yes, regenerate

Method 2: Counter-based
- Use auto-incrementing ID
- Convert number to Base62
```

**Reference:** [Base62 Encoding](https://en.wikipedia.org/wiki/Base62)

---

### Step 3: URL Validation
**Todo:**
- [ ] Create `utils/urlValidator.js`
- [ ] Validate URL format
- [ ] Check protocol (http/https)
- [ ] Prevent malicious URLs
- [ ] Sanitize input

**Hint:** Use regex or URL constructor to validate:
```javascript
new URL(urlString) // throws error if invalid
```

---

### Step 4: Create Routes
**Todo:**
- [ ] `POST /api/shorten` - Create short URL
  - Accept: { originalUrl, customAlias (optional) }
  - Return: { shortUrl, shortCode, originalUrl }
- [ ] `GET /:shortCode` - Redirect to original URL
- [ ] `GET /api/urls` - Get all URLs created
- [ ] `GET /api/urls/:shortCode/stats` - Get analytics
- [ ] `DELETE /api/urls/:shortCode` - Delete short URL

---

### Step 5: Implement Redirect Logic
**Todo:**
- [ ] Find URL by shortCode
- [ ] If found:
  - Increment click count
  - Update lastAccessedAt
  - Redirect with `res.redirect(originalUrl)`
- [ ] If not found:
  - Return 404 with nice error page
- [ ] Use 302 redirect (temporary)

**Hint:** 
```javascript
res.redirect(302, originalUrl);
```

**Why 302?** Allows you to track clicks. 301 (permanent) gets cached by browsers.

---

### Step 6: Analytics Tracking
**Todo:**
- [ ] Track clicks count
- [ ] Track timestamps
- [ ] (Optional) Track referrer
- [ ] (Optional) Track browser/device
- [ ] (Optional) Track location (IP-based)

**Basic Analytics:**
- Total clicks
- Created date
- Last accessed date
- Clicks per day

---

### Step 7: Custom Short Codes
**Todo:**
- [ ] Allow users to choose custom alias
- [ ] Validate: 
  - 3-20 characters
  - Only a-z, A-Z, 0-9, hyphens
  - Not already taken
- [ ] Prevent reserved words (admin, api, stats, etc.)

**Reserved Words to Block:**
```javascript
const reserved = ['admin', 'api', 'stats', 'dashboard', 'login'];
```

---

### Step 8: React Frontend
**Todo:**
- [ ] URL input form
- [ ] Optional custom alias field
- [ ] Display shortened URL
- [ ] Copy to clipboard button
- [ ] QR code generation (optional)
- [ ] Show list of created URLs
- [ ] Display analytics for each URL
- [ ] Delete functionality

**React Integration Example:**
```jsx
import React, { useState } from 'react';

function UrlShortener() {
  const [originalUrl, setOriginalUrl] = useState('');
  const [shortUrl, setShortUrl] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');
    setShortUrl('');

    try {
      const response = await fetch('http://localhost:5000/api/shorten', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ originalUrl }),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Failed to shorten URL');
      }

      const data = await response.json();
      setShortUrl(data.shortUrl);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          type="url"
          value={originalUrl}
          onChange={(e) => setOriginalUrl(e.target.value)}
          placeholder="Enter a long URL"
          required
        />
        <button type="submit" disabled={loading}>
          {loading ? 'Shortening...' : 'Shorten'}
        </button>
      </form>

      {error && <p style={{ color: 'red' }}>{error}</p>}

      {shortUrl && (
        <div>
          <p>Short URL: <a href={shortUrl} target="_blank" rel="noopener noreferrer">{shortUrl}</a></p>
          <button onClick={() => navigator.clipboard.writeText(shortUrl)}>
            Copy
          </button>
        </div>
      )}
    </div>
  );
}

export default UrlShortener;
```

---

## 💡 Hints & Tips

### Base62 Alphabet:
```javascript
const BASE62 = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
```

### Random Code Generation:
```javascript
function generateCode(length = 6) {
  let code = '';
  for (let i = 0; i < length; i++) {
    code += BASE62[Math.floor(Math.random() * 62)];
  }
  return code;
}
```

### Check URL Validity:
```javascript
function isValidUrl(string) {
  try {
    const url = new URL(string);
    return url.protocol === 'http:' || url.protocol === 'https:';
  } catch {
    return false;
  }
}
```

### React Copy to Clipboard:
```javascript
navigator.clipboard.writeText(shortUrl);
```

---

## 🎨 Enhancement Ideas

**Easy:**
- QR code generation
- Expiry dates for URLs
- Password protection
- URL preview before redirect

**Medium:**
- Custom domains
- Bulk URL shortening
- CSV import/export
- Browser extension

**Advanced:**
- User accounts and dashboards
- Team workspaces
- A/B testing URLs
- Deep linking for mobile apps
- Geographic redirects

---

## ✅ Completion Checklist

**Backend:**
- [ ] Short code generation works
- [ ] URL validation implemented
- [ ] Redirect working (302)
- [ ] Click tracking accurate
- [ ] Custom aliases supported
- [ ] Analytics endpoint working
- [ ] No duplicate codes

**Frontend:**
- [ ] URL submission form
- [ ] Shortened URL displayed
- [ ] Copy to clipboard works
- [ ] Analytics visible
- [ ] URL list shows all shortened URLs
- [ ] Delete functionality
- [ ] Error handling

---

## 📚 Concepts You'll Practice

- Hash generation
- Base62 encoding
- HTTP redirects
- URL validation
- Analytics tracking
- Unique ID generation
- Collision handling
- React clipboard API

---

## 🔗 Related Resources

**Chapters:**
- [B02: Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

**External:**
- [URL API](https://developer.mozilla.org/en-US/docs/Web/API/URL)
- [HTTP Redirects](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections)
- [Base62 Encoding](https://stackoverflow.com/questions/742013/how-do-i-create-a-url-shortener)

---

**Next:** [Project 7: Contact Form API](07-contact-form-api.md)
