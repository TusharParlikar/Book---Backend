# Chapter 19: Data Analysis with Python & Pandas 📊

**Analyze Your Application Data for Business Insights**

---

## 📚 What You'll Learn

- Python Pandas for data manipulation
- MongoDB aggregation pipelines
- Data cleaning and preprocessing
- Statistical analysis
- Data visualization with Matplotlib
- Exporting reports (CSV, Excel, PDF)
- Scheduling automated reports
- Building analytics APIs

**Time Required:** 6-8 hours  
**Difficulty:** Advanced  
**Prerequisites:** B01-B17, Basic Python knowledge

---

## 🎯 Learning Objectives

By the end of this chapter, you will:

✅ Query and analyze MongoDB data with Python  
✅ Use Pandas for data manipulation  
✅ Generate business insights and reports  
✅ Create data visualizations  
✅ Export analysis to multiple formats  
✅ Build analytics REST APIs  
✅ Automate report generation  
✅ Optimize query performance for large datasets  

---

## 🐍 Setup Python Analytics Service

### Install Dependencies

```bash
pip install pandas pymongo matplotlib seaborn openpyxl reportlab
```

<details>
<summary>💡 Expected Output</summary>

```bash
Collecting pandas
  Downloading pandas-2.0.1-cp310-cp310-win_amd64.whl (11.6 MB)
...
Successfully installed pandas-2.0.1 pymongo-4.3.3 matplotlib-3.7.1 seaborn-0.12.2 openpyxl-3.1.2 reportlab-3.6.12
```
</details>

### Connect to MongoDB

**File:** `analytics-service/db.py`

```python
from pymongo import MongoClient
import pandas as pd
import os

# Connect to MongoDB
client = MongoClient(os.getenv('MONGODB_URI', 'mongodb://localhost:27017/'))
db = client['your_database']

def get_collection_as_dataframe(collection_name, query={}, projection=None):
    """Fetch MongoDB collection as Pandas DataFrame"""
    collection = db[collection_name]
    cursor = collection.find(query, projection)
    return pd.DataFrame(list(cursor))
```

---

## 📊 E-Commerce Analytics

### Sales Analysis

```python
from flask import Flask, jsonify, request
import pandas as pd
from datetime import datetime, timedelta
from db import get_collection_as_dataframe

app = Flask(__name__)

@app.route('/analytics/sales/summary', methods=['GET'])
def sales_summary():
    # Get date range from query params
    days = int(request.args.get('days', 30))
    start_date = datetime.now() - timedelta(days=days)

    # Fetch orders from MongoDB
    orders_df = get_collection_as_dataframe('orders', {
        'createdAt': {'$gte': start_date},
        'status': 'paid'
    })

    if orders_df.empty:
        return jsonify({'message': 'No data found'}), 404

    # Calculate metrics
    total_revenue = orders_df['totalAmount'].sum()
    total_orders = len(orders_df)
    average_order_value = orders_df['totalAmount'].mean()
    
    # Daily revenue
    orders_df['date'] = pd.to_datetime(orders_df['createdAt']).dt.date
    daily_revenue = orders_df.groupby('date')['totalAmount'].sum().to_dict()

    return jsonify({
        'period': f'Last {days} days',
        'total_revenue': float(total_revenue),
        'total_orders': total_orders,
        'average_order_value': float(average_order_value),
        'daily_revenue': {str(k): float(v) for k, v in daily_revenue.items()}
    })
```

### Product Performance

```python
@app.route('/analytics/products/top', methods=['GET'])
def top_products():
    limit = int(request.args.get('limit', 10))
    
    # Fetch all orders
    orders_df = get_collection_as_dataframe('orders', {'status': 'paid'})
    
    # Explode items array
    items_list = []
    for _, order in orders_df.iterrows():
        for item in order['items']:
            items_list.append({
                'product_id': str(item['product']),
                'quantity': item['quantity'],
                'revenue': item['price'] * item['quantity']
            })
    
    items_df = pd.DataFrame(items_list)
    
    # Aggregate by product
    product_stats = items_df.groupby('product_id').agg({
        'quantity': 'sum',
        'revenue': 'sum'
    }).reset_index()
    
    # Sort by revenue
    product_stats = product_stats.sort_values('revenue', ascending=False).head(limit)
    
    # Fetch product names
    from bson import ObjectId
    product_ids = [ObjectId(pid) for pid in product_stats['product_id']]
    products_df = get_collection_as_dataframe('products', {
        '_id': {'$in': product_ids}
    })
    
    # Merge
    product_stats['_id'] = product_stats['product_id'].apply(lambda x: ObjectId(x))
    result = product_stats.merge(
        products_df[['_id', 'name']], 
        on='_id'
    )
    
    top_products = result[['name', 'quantity', 'revenue']].to_dict('records')
    
    return jsonify({
        'top_products': top_products
    })
```

### Customer Analytics

```python
@app.route('/analytics/customers/segments', methods=['GET'])
def customer_segments():
    # Fetch users and their orders
    orders_df = get_collection_as_dataframe('orders', {'status': 'paid'})
    
    # Calculate RFM (Recency, Frequency, Monetary)
    current_date = datetime.now()
    
    customer_stats = orders_df.groupby('user').agg({
        'createdAt': lambda x: (current_date - pd.to_datetime(x.max())).days,  # Recency
        '_id': 'count',  # Frequency
        'totalAmount': 'sum'  # Monetary
    }).reset_index()
    
    customer_stats.columns = ['user', 'recency', 'frequency', 'monetary']
    
    # Segment customers
    def segment_customer(row):
        if row['monetary'] > 50000 and row['frequency'] > 10:
            return 'VIP'
        elif row['monetary'] > 20000 and row['frequency'] > 5:
            return 'Loyal'
        elif row['recency'] < 30:
            return 'Active'
        elif row['recency'] > 90:
            return 'At Risk'
        else:
            return 'Regular'
    
    customer_stats['segment'] = customer_stats.apply(segment_customer, axis=1)
    
    # Count segments
    segments = customer_stats['segment'].value_counts().to_dict()
    
    # Average values per segment
    segment_stats = customer_stats.groupby('segment').agg({
        'recency': 'mean',
        'frequency': 'mean',
        'monetary': 'mean'
    }).to_dict('index')
    
    return jsonify({
        'segment_counts': segments,
        'segment_averages': segment_stats
    })
```

---

## 📈 Data Visualization

### Generate Charts

```python
import matplotlib
matplotlib.use('Agg')  # Non-GUI backend
import matplotlib.pyplot as plt
import seaborn as sns
import io
import base64

@app.route('/analytics/charts/revenue-trend', methods=['GET'])
def revenue_trend_chart():
    days = int(request.args.get('days', 30))
    start_date = datetime.now() - timedelta(days=days)
    
    # Fetch data
    orders_df = get_collection_as_dataframe('orders', {
        'createdAt': {'$gte': start_date},
        'status': 'paid'
    })
    
    orders_df['date'] = pd.to_datetime(orders_df['createdAt']).dt.date
    daily_revenue = orders_df.groupby('date')['totalAmount'].sum().reset_index()
    
    # Create plot
    plt.figure(figsize=(12, 6))
    plt.plot(daily_revenue['date'], daily_revenue['totalAmount'], marker='o')
    plt.title(f'Revenue Trend - Last {days} Days')
    plt.xlabel('Date')
    plt.ylabel('Revenue (₹)')
    plt.xticks(rotation=45)
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    
    # Convert to base64
    buffer = io.BytesIO()
    plt.savefig(buffer, format='png', dpi=150)
    buffer.seek(0)
    image_base64 = base64.b64encode(buffer.read()).decode()
    plt.close()
    
    return jsonify({
        'chart': f'data:image/png;base64,{image_base64}'
    })

@app.route('/analytics/charts/category-distribution', methods=['GET'])
def category_distribution():
    # Fetch products with categories
    products_df = get_collection_as_dataframe('products', {}, {'category': 1})
    
    # Count by category
    category_counts = products_df['category'].value_counts()
    
    # Create pie chart
    plt.figure(figsize=(10, 8))
    plt.pie(category_counts.values, labels=category_counts.index, autopct='%1.1f%%')
    plt.title('Products by Category')
    
    # Convert to base64
    buffer = io.BytesIO()
    plt.savefig(buffer, format='png', dpi=150)
    buffer.seek(0)
    image_base64 = base64.b64encode(buffer.read()).decode()
    plt.close()
    
    return jsonify({
        'chart': f'data:image/png;base64,{image_base64}'
    })
```

---

## 📄 Export Reports

### Generate Excel Report

```python
from openpyxl.styles import Font, Alignment, PatternFill
from openpyxl.chart import BarChart, Reference
from flask import send_file

@app.route('/analytics/reports/excel', methods=['GET'])
def generate_excel_report():
    # Fetch data
    orders_df = get_collection_as_dataframe('orders', {'status': 'paid'})
    
    # Create Excel writer
    output = io.BytesIO()
    with pd.ExcelWriter(output, engine='openpyxl') as writer:
        # Sheet 1: Summary
        summary_data = {
            'Metric': ['Total Orders', 'Total Revenue', 'Average Order Value'],
            'Value': [
                len(orders_df),
                orders_df['totalAmount'].sum(),
                orders_df['totalAmount'].mean()
            ]
        }
        summary_df = pd.DataFrame(summary_data)
        summary_df.to_excel(writer, sheet_name='Summary', index=False)
        
        # Sheet 2: Daily Revenue
        orders_df['date'] = pd.to_datetime(orders_df['createdAt']).dt.date
        daily_revenue = orders_df.groupby('date')['totalAmount'].sum().reset_index()
        daily_revenue.columns = ['Date', 'Revenue']
        daily_revenue.to_excel(writer, sheet_name='Daily Revenue', index=False)
        
        # Sheet 3: All Orders
        orders_export = orders_df[['_id', 'totalAmount', 'status', 'createdAt']]
        orders_export.to_excel(writer, sheet_name='Orders', index=False)
    
    output.seek(0)
    
    return send_file(
        output,
        mimetype='application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
        as_attachment=True,
        download_name=f'sales_report_{datetime.now().strftime("%Y%m%d")}.xlsx'
    )
```

### Generate PDF Report

```python
from reportlab.lib.pagesizes import letter, A4
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib import colors

@app.route('/analytics/reports/pdf', methods=['GET'])
def generate_pdf_report():
    output = io.BytesIO()
    
    # Create PDF
    doc = SimpleDocTemplate(output, pagesize=A4)
    elements = []
    styles = getSampleStyleSheet()
    
    # Title
    title = Paragraph("Sales Analytics Report", styles['Title'])
    elements.append(title)
    elements.append(Spacer(1, 20))
    
    # Fetch data
    orders_df = get_collection_as_dataframe('orders', {'status': 'paid'})
    
    # Summary table
    summary_data = [
        ['Metric', 'Value'],
        ['Total Orders', str(len(orders_df))],
        ['Total Revenue', f"₹{orders_df['totalAmount'].sum():,.2f}"],
        ['Average Order Value', f"₹{orders_df['totalAmount'].mean():,.2f}"]
    ]
    
    summary_table = Table(summary_data)
    summary_table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
        ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
        ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
        ('GRID', (0, 0), (-1, -1), 1, colors.black)
    ]))
    
    elements.append(summary_table)
    elements.append(Spacer(1, 30))
    
    # Top products
    products_heading = Paragraph("Top 10 Products", styles['Heading2'])
    elements.append(products_heading)
    elements.append(Spacer(1, 12))
    
    # Build PDF
    doc.build(elements)
    output.seek(0)
    
    return send_file(
        output,
        mimetype='application/pdf',
        as_attachment=True,
        download_name=f'analytics_report_{datetime.now().strftime("%Y%m%d")}.pdf'
    )
```

---

## 🤖 Automated Reports

### Schedule with Celery

**Install:**

```bash
pip install celery redis
```

**File:** `tasks.py`

```python
from celery import Celery
from celery.schedules import crontab
import pandas as pd
from db import get_collection_as_dataframe
from datetime import datetime, timedelta

app = Celery('analytics', broker='redis://localhost:6379/0')

@app.task
def generate_daily_report():
    """Generate daily sales report and email to admin"""
    yesterday = datetime.now() - timedelta(days=1)
    
    # Fetch yesterday's orders
    orders_df = get_collection_as_dataframe('orders', {
        'createdAt': {
            '$gte': yesterday.replace(hour=0, minute=0, second=0),
            '$lt': yesterday.replace(hour=23, minute=59, second=59)
        },
        'status': 'paid'
    })
    
    # Calculate metrics
    total_revenue = orders_df['totalAmount'].sum()
    total_orders = len(orders_df)
    
    # Create Excel report
    output = io.BytesIO()
    with pd.ExcelWriter(output, engine='openpyxl') as writer:
        orders_df.to_excel(writer, sheet_name='Orders', index=False)
    
    # Send email (using your email service)
    # send_email(
    #     to='admin@example.com',
    #     subject=f'Daily Sales Report - {yesterday.strftime("%Y-%m-%d")}',
    #     body=f'Total Revenue: ₹{total_revenue}\nTotal Orders: {total_orders}',
    #     attachment=output.getvalue()
    # )
    
    return f'Report generated: ₹{total_revenue}, {total_orders} orders'

# Schedule tasks
app.conf.beat_schedule = {
    'daily-report': {
        'task': 'tasks.generate_daily_report',
        'schedule': crontab(hour=9, minute=0),  # Every day at 9 AM
    },
}
```

**Run Celery:**

```bash
celery -A tasks beat --loglevel=info
celery -A tasks worker --loglevel=info
```

---

## 🔗 Node.js Integration

### Call Python Analytics from Node.js

```javascript
const axios = require('axios');

const ANALYTICS_SERVICE_URL = process.env.ANALYTICS_SERVICE_URL || 'http://localhost:5003';

// Get sales summary
router.get('/admin/analytics/sales', auth, isAdmin, async (req, res) => {
  try {
    const { days = 30 } = req.query;
    
    const response = await axios.get(
      `${ANALYTICS_SERVICE_URL}/analytics/sales/summary?days=${days}`
    );
    
    res.json(response.data);
    
  } catch (error) {
    console.error('Analytics service error:', error);
    res.status(500).json({ message: 'Failed to fetch analytics' });
  }
});

// Download Excel report
router.get('/admin/reports/excel', auth, isAdmin, async (req, res) => {
  try {
    const response = await axios.get(
      `${ANALYTICS_SERVICE_URL}/analytics/reports/excel`,
      { responseType: 'arraybuffer' }
    );
    
    res.setHeader('Content-Type', 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
    res.setHeader('Content-Disposition', `attachment; filename=sales_report_${Date.now()}.xlsx`);
    res.send(response.data);
    
  } catch (error) {
    console.error('Report generation error:', error);
    res.status(500).json({ message: 'Failed to generate report' });
  }
});
```

---

## 📊 React Analytics Dashboard

```jsx
import { useState, useEffect } from 'react';
import { Line, Pie, Bar } from 'react-chartjs-2';

function AnalyticsDashboard() {
  const [summary, setSummary] = useState(null);
  const [chartData, setChartData] = useState(null);

  useEffect(() => {
    fetchAnalytics();
  }, []);

  const fetchAnalytics = async () => {
    const response = await fetch('/api/admin/analytics/sales?days=30', {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });
    const data = await response.json();
    setSummary(data);

    // Prepare chart data
    setChartData({
      labels: Object.keys(data.daily_revenue),
      datasets: [{
        label: 'Daily Revenue',
        data: Object.values(data.daily_revenue),
        borderColor: 'rgb(75, 192, 192)',
        backgroundColor: 'rgba(75, 192, 192, 0.2)',
      }]
    });
  };

  const downloadExcel = async () => {
    const response = await fetch('/api/admin/reports/excel', {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });
    const blob = await response.blob();
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `sales_report_${Date.now()}.xlsx`;
    a.click();
  };

  if (!summary) return <div>Loading...</div>;

  return (
    <div className="analytics-dashboard">
      <h1>Analytics Dashboard</h1>

      {/* Summary Cards */}
      <div className="metrics-grid">
        <div className="metric-card">
          <h3>Total Revenue</h3>
          <p className="metric-value">₹{summary.total_revenue.toLocaleString()}</p>
        </div>
        <div className="metric-card">
          <h3>Total Orders</h3>
          <p className="metric-value">{summary.total_orders}</p>
        </div>
        <div className="metric-card">
          <h3>Average Order Value</h3>
          <p className="metric-value">₹{summary.average_order_value.toFixed(2)}</p>
        </div>
      </div>

      {/* Revenue Trend Chart */}
      {chartData && (
        <div className="chart-container">
          <h2>Revenue Trend</h2>
          <Line data={chartData} />
        </div>
      )}

      {/* Export Buttons */}
      <div className="export-section">
        <button onClick={downloadExcel}>Download Excel Report</button>
        <button onClick={() => window.open('/api/admin/reports/pdf')}>
          Download PDF Report
        </button>
      </div>
    </div>
  );
}
```

---

## 🎓 Hands-On Exercise

**Build: Complete Analytics Dashboard**

1. **Sales Analytics**:
   - Revenue by day/week/month
   - Order count trends
   - Average order value

2. **Product Analytics**:
   - Top selling products
   - Category performance
   - Stock analysis

3. **Customer Analytics**:
   - New vs returning customers
   - Customer lifetime value
   - RFM segmentation

4. **Reports**:
   - Automated daily email reports
   - Excel export with charts
   - PDF monthly summary

5. **Visualizations**:
   - Line charts for trends
   - Pie charts for distribution
   - Bar charts for comparisons

---

## 📚 Practice Projects

- **Project #40:** [Analytics Dashboard](../projects/40-analytics-dashboard.md) ⭐
- **Project #55:** [Business Intelligence Platform](../projects/55-business-intelligence.md) ⭐

---

## ✅ Chapter Checklist

- [ ] Set up Python analytics service
- [ ] Connect to MongoDB with Pandas
- [ ] Build sales analytics endpoints
- [ ] Create data visualizations
- [ ] Generate Excel reports
- [ ] Generate PDF reports
- [ ] Schedule automated reports
- [ ] Build React analytics dashboard
- [ ] Integrate with Node.js backend

---

**Made with ❤️ by Tushar Parlikar**

[← Previous: ML Model Integration](B18_ML_MODEL_INTEGRATION.md) | [Next: Real-Time Analytics →](B20_REALTIME_ANALYTICS.md)
