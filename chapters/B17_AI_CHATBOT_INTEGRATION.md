# Chapter 17: AI Chatbot Integration 🤖

**Build Intelligent Conversational AI into Your Applications**

---

## 📚 What You'll Learn

- Integrating OpenAI GPT models
- Using Google Gemini API
- Streaming chat responses in real-time
- Managing conversation history
- Function calling with AI
- Building context-aware chatbots
- Rate limiting and cost management
- Creating AI-powered customer support

**Time Required:** 8-10 hours  
**Difficulty:** Advanced  
**Prerequisites:** B01-B09 (Backend basics, Auth, Database)

---

## 🎯 Learning Objectives

By the end of this chapter, you will:

✅ Integrate OpenAI GPT-4 and GPT-3.5  
✅ Use Google Gemini for conversational AI  
✅ Stream responses in real-time to frontend  
✅ Store and manage conversation history  
✅ Implement function calling for dynamic actions  
✅ Build RAG (Retrieval Augmented Generation) systems  
✅ Optimize API costs with caching and rate limiting  
✅ Create production-ready AI chatbots  

---

## 🤖 AI Models Overview

### OpenAI GPT Models

| Model | Best For | Cost | Speed |
|-------|----------|------|-------|
| **GPT-4 Turbo** | Complex reasoning, coding | High | Slower |
| **GPT-3.5 Turbo** | General chat, fast responses | Low | Fast |
| **GPT-4o** | Multimodal (text + images) | Medium | Medium |

### Google Gemini Models

| Model | Best For | Cost | Speed |
|-------|----------|------|-------|
| **Gemini 1.5 Pro** | Long context, complex tasks | Medium | Medium |
| **Gemini 1.5 Flash** | Fast responses, high throughput | Low | Very Fast |

**Recommendation:** Use **GPT-3.5 Turbo** or **Gemini Flash** for production chatbots (cost-effective).

---

## 🔧 Setup: OpenAI Integration

### Step 1: Get API Key

1. Go to [platform.openai.com](https://platform.openai.com/)
2. Create account and add payment method
3. Generate API key
4. Copy key to `.env`

### Step 2: Install SDK

```bash
npm install openai
```

<details>
<summary>💡 Expected Output</summary>

```bash
added 1 package, and audited 2 packages in 1s
found 0 vulnerabilities
```
</details>

### Step 3: Environment Variables

```env
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxx
```

### Step 4: Initialize Client

Create `utils/openai.js`:

```javascript
const OpenAI = require('openai');

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

module.exports = openai;
```

---

## 💬 Basic Chat Completion

### Simple Chat Endpoint

**Route:** `POST /api/chat`

```javascript
const openai = require('../utils/openai');

router.post('/chat', async (req, res) => {
  const { message } = req.body;

  try {
    const completion = await openai.chat.completions.create({
      model: 'gpt-3.5-turbo',
      messages: [
        {
          role: 'system',
          content: 'You are a helpful assistant for an e-commerce website.'
        },
        {
          role: 'user',
          content: message
        }
      ],
      temperature: 0.7, // Creativity (0-2)
      max_tokens: 500, // Max response length
    });

    const reply = completion.choices[0].message.content;

    res.json({
      success: true,
      reply,
      usage: completion.usage // Track token usage
    });

  } catch (error) {
    console.error('OpenAI error:', error);
    res.status(500).json({ message: 'AI request failed' });
  }
});
```

---

## 🌊 Streaming Responses

Stream AI responses token-by-token for better UX.

### Backend: Server-Sent Events (SSE)

```javascript
router.post('/chat/stream', async (req, res) => {
  const { message } = req.body;

  // Set headers for SSE
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  try {
    const stream = await openai.chat.completions.create({
      model: 'gpt-3.5-turbo',
      messages: [
        { role: 'system', content: 'You are a helpful assistant.' },
        { role: 'user', content: message }
      ],
      stream: true, // Enable streaming
    });

    // Stream tokens to client
    for await (const chunk of stream) {
      const content = chunk.choices[0]?.delta?.content || '';
      
      if (content) {
        res.write(`data: ${JSON.stringify({ content })}\n\n`);
      }
    }

    res.write('data: [DONE]\n\n');
    res.end();

  } catch (error) {
    res.write(`data: ${JSON.stringify({ error: 'Failed' })}\n\n`);
    res.end();
  }
});
```

### Frontend: React with EventSource

```jsx
const [messages, setMessages] = useState([]);
const [currentReply, setCurrentReply] = useState('');

const sendMessage = async (userMessage) => {
  setMessages(prev => [...prev, { role: 'user', content: userMessage }]);
  setCurrentReply('...'); // Show loading indicator

  try {
    const response = await fetch('/api/chat/stream', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: userMessage })
    });

    if (!response.ok || !response.body) {
      throw new Error('Failed to get stream response.');
    }

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let aiReply = '';

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value, { stream: true });
      const lines = chunk.split('\n\n');

      for (const line of lines) {
        if (line.startsWith('data: ')) {
          const data = line.slice(6);
          
          if (data.trim() === '[DONE]') {
            setMessages(prev => [...prev, { role: 'assistant', content: aiReply }]);
            setCurrentReply('');
            return; // Exit loop
          }
          
          try {
            const parsed = JSON.parse(data);
            if (parsed.content) {
              aiReply += parsed.content;
              setCurrentReply(aiReply);
            }
          } catch (e) {
            // Ignore JSON parsing errors for incomplete chunks
          }
        }
      }
    }
  } catch (error) {
    console.error('Stream error:', error);
    setCurrentReply('Error: Could not get response.');
  }
};
```

---

## 💾 Conversation History Management

### Database Schema

```javascript
const conversationSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  title: {
    type: String,
    default: 'New Chat'
  },
  messages: [{
    role: {
      type: String,
      enum: ['system', 'user', 'assistant'],
      required: true
    },
    content: {
      type: String,
      required: true
    },
    timestamp: {
      type: Date,
      default: Date.now
    }
  }],
  model: {
    type: String,
    default: 'gpt-3.5-turbo'
  },
  tokenCount: {
    type: Number,
    default: 0
  }
}, { timestamps: true });
```

### Create New Conversation

```javascript
router.post('/conversations', auth, async (req, res) => {
  const conversation = await Conversation.create({
    user: req.user._id,
    messages: [{
      role: 'system',
      content: 'You are a helpful AI assistant.'
    }]
  });

  res.json({ success: true, conversation });
});
```

### Send Message with History

```javascript
router.post('/conversations/:id/message', auth, async (req, res) => {
  const { message } = req.body;
  const conversation = await Conversation.findById(req.params.id);

  if (!conversation || conversation.user.toString() !== req.user._id.toString()) {
    return res.status(404).json({ message: 'Conversation not found' });
  }

  // Add user message to history
  conversation.messages.push({
    role: 'user',
    content: message
  });

  // Send entire conversation history to OpenAI
  const completion = await openai.chat.completions.create({
    model: conversation.model,
    messages: conversation.messages.map(msg => ({
      role: msg.role,
      content: msg.content
    })),
    max_tokens: 500
  });

  const aiReply = completion.choices[0].message.content;

  // Save AI response
  conversation.messages.push({
    role: 'assistant',
    content: aiReply
  });

  // Update token count
  conversation.tokenCount += completion.usage.total_tokens;
  
  // Auto-generate title from first message
  if (conversation.messages.length === 3) { // system + user + assistant
    conversation.title = message.slice(0, 50) + '...';
  }

  await conversation.save();

  res.json({
    success: true,
    reply: aiReply,
    tokensUsed: completion.usage.total_tokens
  });
});
```

---

## 🔧 Google Gemini Integration

### Setup

```bash
npm install @google/generative-ai
```

```env
GEMINI_API_KEY=your_gemini_api_key
```

### Initialize Gemini

Create `utils/gemini.js`:

```javascript
const { GoogleGenerativeAI } = require('@google/generative-ai');

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);

const model = genAI.getGenerativeModel({ 
  model: 'gemini-1.5-flash' 
});

module.exports = { model };
```

### Chat with Gemini

```javascript
const { model } = require('../utils/gemini');

router.post('/chat/gemini', async (req, res) => {
  const { message } = req.body;

  try {
    const chat = model.startChat({
      history: [
        {
          role: 'user',
          parts: [{ text: 'You are a helpful assistant.' }]
        },
        {
          role: 'model',
          parts: [{ text: 'Understood! How can I help you today?' }]
        }
      ],
    });

    const result = await chat.sendMessage(message);
    const reply = result.response.text();

    res.json({ success: true, reply });

  } catch (error) {
    console.error('Gemini error:', error);
    res.status(500).json({ message: 'AI request failed' });
  }
});
```

### Streaming with Gemini

```javascript
router.post('/chat/gemini/stream', async (req, res) => {
  const { message } = req.body;

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');

  try {
    const result = await model.generateContentStream(message);

    for await (const chunk of result.stream) {
      const text = chunk.text();
      res.write(`data: ${JSON.stringify({ content: text })}\n\n`);
    }

    res.write('data: [DONE]\n\n');
    res.end();

  } catch (error) {
    res.write(`data: ${JSON.stringify({ error: 'Failed' })}\n\n`);
    res.end();
  }
});
```

---

## ⚡ Function Calling

Allow AI to call backend functions dynamically.

### Define Functions

```javascript
const functions = [
  {
    name: 'get_weather',
    description: 'Get current weather for a location',
    parameters: {
      type: 'object',
      properties: {
        location: {
          type: 'string',
          description: 'City name, e.g. Mumbai'
        }
      },
      required: ['location']
    }
  },
  {
    name: 'get_user_orders',
    description: 'Get user\'s recent orders',
    parameters: {
      type: 'object',
      properties: {
        limit: {
          type: 'number',
          description: 'Number of orders to fetch'
        }
      }
    }
  }
];
```

### Implement Function Handlers

```javascript
async function executeFunction(functionName, args, userId) {
  try {
    switch (functionName) {
      case 'get_weather':
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${args.location}&appid=${process.env.WEATHER_API_KEY}&units=metric`
        );
        if (!response.ok) throw new Error('Weather API request failed');
        const data = await response.json();
        return `Weather in ${args.location}: ${data.weather[0].description}, ${data.main.temp}°C`;

      case 'get_user_orders':
        const orders = await Order.find({ user: userId })
          .limit(args.limit || 5)
          .sort({ createdAt: -1 });
        if (orders.length === 0) return 'No recent orders found.';
        return orders.map(o => `Order #${o._id}: ${o.status}, ₹${o.totalAmount}`).join('\n');

      default:
        return 'Function not found';
    }
  } catch (error) {
    console.error(`Error executing function ${functionName}:`, error);
    return `Error getting data for ${functionName}.`;
  }
}
```

### Chat with Function Calling

```javascript
router.post('/chat/functions', auth, async (req, res) => {
  const { message } = req.body;

  try {
    let messages = [
      { role: 'system', content: 'You are a helpful assistant.' },
      { role: 'user', content: message }
    ];

    // First API call
    let completion = await openai.chat.completions.create({
      model: 'gpt-3.5-turbo',
      messages,
      functions,
      function_call: 'auto'
    });

    let responseMessage = completion.choices[0].message;

    // Loop to handle multiple function calls if necessary
    while (responseMessage.function_call) {
      const functionName = responseMessage.function_call.name;
      const functionArgs = JSON.parse(responseMessage.function_call.arguments);

      // Execute the function
      const functionResult = await executeFunction(functionName, functionArgs, req.user._id);

      // Add results to conversation history
      messages.push(responseMessage); // AI's decision to call the function
      messages.push({
        role: 'function',
        name: functionName,
        content: functionResult
      });

      // Second API call with function result
      completion = await openai.chat.completions.create({
        model: 'gpt-3.5-turbo',
        messages,
        functions,
        function_call: 'auto'
      });

      responseMessage = completion.choices[0].message;
    }

    res.json({
      success: true,
      reply: responseMessage.content
    });
  } catch (error) {
    console.error('Function calling error:', error);
    res.status(500).json({ success: false, message: 'An error occurred.' });
  }
});
```

**Example:**

```
User: "What's the weather in Mumbai?"
  ↓
AI calls: get_weather({ location: "Mumbai" })
  ↓
Function returns: "Weather in Mumbai: clear sky, 28°C"
  ↓
AI responds: "The weather in Mumbai is currently clear with a temperature of 28°C."
```

---

## 🎯 RAG (Retrieval Augmented Generation)

Give AI access to your custom knowledge base.

### Step 1: Create Embeddings

```bash
npm install pdf-parse
```

```javascript
const openai = require('../utils/openai');
const PDFParser = require('pdf-parse');

// Extract text from PDF
const pdfBuffer = fs.readFileSync('product-catalog.pdf');
const pdfData = await PDFParser(pdfBuffer);
const text = pdfData.text;

// Split into chunks
const chunks = text.match(/.{1,1000}/g); // 1000 char chunks

// Create embeddings for each chunk
const embeddings = [];
for (const chunk of chunks) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: chunk
  });

  embeddings.push({
    text: chunk,
    embedding: response.data[0].embedding
  });
}

// Save to database
await KnowledgeBase.insertMany(embeddings.map(e => ({
  content: e.text,
  embedding: e.embedding
})));
```

### Step 2: Search with Query Embedding

```javascript
// Create embedding for user query
const queryEmbedding = await openai.embeddings.create({
  model: 'text-embedding-3-small',
  input: userQuery
});

// Find similar chunks (cosine similarity)
const results = await KnowledgeBase.aggregate([
  {
    $addFields: {
      similarity: {
        // Vector similarity calculation
        $divide: [
          {
            $reduce: {
              input: { $range: [0, { $size: '$embedding' }] },
              initialValue: 0,
              in: {
                $add: [
                  '$$value',
                  {
                    $multiply: [
                      { $arrayElemAt: ['$embedding', '$$this'] },
                      { $arrayElemAt: [queryEmbedding.data[0].embedding, '$$this'] }
                    ]
                  }
                ]
              }
            }
          },
          1
        ]
      }
    }
  },
  { $sort: { similarity: -1 } },
  { $limit: 3 }
]);

// Use top results as context
const context = results.map(r => r.content).join('\n\n');
```

### Step 3: Chat with Context

```javascript
const completion = await openai.chat.completions.create({
  model: 'gpt-3.5-turbo',
  messages: [
    {
      role: 'system',
      content: `You are a helpful assistant. Use this context to answer questions:\n\n${context}`
    },
    {
      role: 'user',
      content: userQuery
    }
  ]
});
```

---

## 💰 Cost Management

### Track Token Usage

```javascript
const usageSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  model: String,
  tokensUsed: Number,
  cost: Number, // In USD
  date: { type: Date, default: Date.now }
});

// After each API call
await Usage.create({
  user: req.user._id,
  model: 'gpt-3.5-turbo',
  tokensUsed: completion.usage.total_tokens,
  cost: (completion.usage.total_tokens / 1000) * 0.002 // $0.002 per 1K tokens
});
```

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const chatLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 10, // 10 requests per minute
  message: 'Too many chat requests. Please wait.'
});

router.use('/chat', chatLimiter);
```

### Response Caching

```javascript
const redis = require('../utils/redis');

router.post('/chat', async (req, res) => {
  const { message } = req.body;

  // Check cache
  const cacheKey = `chat:${message.toLowerCase()}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    return res.json({ success: true, reply: cached, cached: true });
  }

  // Call OpenAI
  const completion = await openai.chat.completions.create({
    model: 'gpt-3.5-turbo',
    messages: [{ role: 'user', content: message }]
  });

  const reply = completion.choices[0].message.content;

  // Cache for 1 hour
  await redis.setex(cacheKey, 3600, reply);

  res.json({ success: true, reply, cached: false });
});
```

---

## 🎨 React Chat UI

```jsx
import { useState, useRef, useEffect } from 'react';

function ChatInterface() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [loading, setLoading] = useState(false);
  const messagesEndRef = useRef(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(scrollToBottom, [messages]);

  const sendMessage = async () => {
    if (!input.trim()) return;

    const userMessage = { role: 'user', content: input };
    setMessages(prev => [...prev, userMessage]);
    setInput('');
    setLoading(true);

    try {
      const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: input })
      });

      const data = await response.json();
      
      setMessages(prev => [...prev, {
        role: 'assistant',
        content: data.reply
      }]);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="chat-container">
      <div className="messages">
        {messages.map((msg, i) => (
          <div key={i} className={`message ${msg.role}`}>
            <div className="avatar">
              {msg.role === 'user' ? '👤' : '🤖'}
            </div>
            <div className="content">{msg.content}</div>
          </div>
        ))}
        {loading && <div className="loading">AI is typing...</div>}
        <div ref={messagesEndRef} />
      </div>

      <div className="input-area">
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
          placeholder="Type your message..."
        />
        <button onClick={sendMessage} disabled={loading}>
          Send
        </button>
      </div>
    </div>
  );
}
```

---

## 🎓 Hands-On Exercise

**Build: AI Customer Support System**

Create an intelligent customer support chatbot:

1. **Features**:
   - Chat interface with conversation history
   - Function calling to check order status, track shipments
   - RAG integration with product documentation
   - Escalate to human agent if needed
   - Multi-language support
   - Sentiment analysis

2. **Backend**:
   - Store conversations in MongoDB
   - Implement function calling for order lookup
   - Create embeddings for product docs
   - Rate limiting per user
   - Track AI costs per conversation

3. **Frontend**:
   - Real-time streaming responses
   - Markdown rendering for AI replies
   - File upload for context (images, PDFs)
   - Export conversation as PDF

**Deliverables:**
- Full-stack chatbot with React + Express
- Function calling for dynamic actions
- RAG system with product knowledge
- Cost tracking dashboard

---

## 📚 Practice Projects

After this chapter, build:

- **Project #52:** [AI-Powered E-Commerce Assistant](../projects/52-ai-ecommerce-assistant.md) ⭐
- **Project #53:** [AI Code Review Bot](../projects/53-ai-code-reviewer.md) ⭐
- **Project #54:** [AI Content Generator](../projects/54-ai-content-generator.md)

---

## 🔗 Resources

**OpenAI:**
- [API Documentation](https://platform.openai.com/docs)
- [Function Calling Guide](https://platform.openai.com/docs/guides/function-calling)
- [Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)

**Google Gemini:**
- [Gemini API Docs](https://ai.google.dev/docs)
- [Quickstart Guide](https://ai.google.dev/tutorials/quickstart)

---

## ✅ Chapter Checklist

- [ ] Set up OpenAI and Gemini API keys
- [ ] Create basic chat completion endpoint
- [ ] Implement streaming responses
- [ ] Store conversation history in database
- [ ] Build function calling system
- [ ] Create RAG system with embeddings
- [ ] Implement rate limiting and caching
- [ ] Track AI costs and token usage
- [ ] Build React chat interface
- [ ] Test with different AI models

---

## 🎯 What's Next?

Next chapter: **Machine Learning Model Integration** - Deploy ML models, run predictions, and build recommendation systems!

---

**Made with ❤️ by Tushar Parlikar**

[← Previous: Razorpay Payment Integration](B16_RAZORPAY_PAYMENT_INTEGRATION.md) | [Next: ML Model Integration →](B18_ML_MODEL_INTEGRATION.md)
