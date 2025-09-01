Social Media Dashboard Starter

   

A starter full-stack project for building a simple social media dashboard.

Backend: Node.js + Express + PostgreSQL

Frontend: React + Vite + Recharts

Features: View metrics, schedule posts, visualize impressions

code

social-media-dashboard/
│── backend/
│   ├── server.js
│   ├── package.json
│   ├── .env.example
│
│── frontend/
│   ├── index.html
│   ├── package.json
│   ├── .env.example
│   └── src/
│       ├── main.jsx
│       └── App.jsx


---

2. Backend Files

backend/package.json

{
  "name": "smd-backend",
  "version": "0.1.0",
  "type": "module",
  "main": "server.js",
  "scripts": {
    "dev": "node server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "morgan": "^1.10.0",
    "pg": "^8.11.5"
  }
}

backend/.env.example

PORT=5002
DATABASE_URL=postgres://postgres:postgres@localhost:5432/smd_dev

backend/server.js

import express from 'express';
import cors from 'cors';
import morgan from 'morgan';
import dotenv from 'dotenv';
import { Pool } from 'pg';

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());
app.use(morgan('dev'));

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

app.get('/', (req, res) => res.json({ status: 'ok', service: 'smd-backend' }));

// Minimal schema init endpoint (dev convenience)
app.post('/api/dev/init', async (req, res) => {
  await pool.query(`
    create table if not exists posts(
      id serial primary key,
      platform text,
      content text,
      created_at timestamptz default now()
    );
    create table if not exists metrics(
      id serial primary key,
      platform text,
      metric text,
      value int,
      captured_at timestamptz default now()
    );
  `);
  res.json({ message: 'initialized' });
});

// Mock metrics
app.get('/api/metrics/summary', async (req, res) => {
  res.json({
    twitter: { followers: 1234, impressions: 9876 },
    instagram: { followers: 2345, impressions: 7654 }
  });
});

// Schedule (mock)
app.post('/api/schedule', async (req, res) => {
  const { platform, content } = req.body;
  const { rows } = await pool.query(
    'insert into posts(platform, content) values ($1,$2) returning *',
    [platform, content]
  );
  res.status(201).json(rows[0]);
});

const PORT = process.env.PORT || 5002;
app.listen(PORT, () => console.log(`SMD API running on port ${PORT}`));


---

3. Frontend Files

frontend/package.json

{
  "name": "smd-frontend",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "recharts": "^2.10.3"
  },
  "devDependencies": {
    "vite": "^5.2.0"
  }
}

frontend/.env.example

VITE_API_URL=http://localhost:5002

frontend/index.html

<!doctype html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Social Media Dashboard</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>

frontend/src/main.jsx

import React from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(<App />)

frontend/src/App.jsx

import React, { useEffect, useState } from 'react'
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts'

const API = import.meta.env.VITE_API_URL || 'http://localhost:5002';

export default function App() {
  const [data, setData] = useState(null);
  const [content, setContent] = useState('Hello world!');
  const [platform, setPlatform] = useState('twitter');

  useEffect(() => {
    fetch(`${API}/api/metrics/summary`).then(r=>r.json()).then(setData).catch(console.error);
  }, []);

  const schedule = async () => {
    await fetch(`${API}/api/schedule`, {
      method:'POST',
      headers:{ 'Content-Type':'application/json' },
      body: JSON.stringify({ platform, content })
    });
    alert('Scheduled locally (mock).');
  };

  const chartData = [
    { name: 'Mon', twitter: 1200, instagram: 900 },
    { name: 'Tue', twitter: 1600, instagram: 1100 },
    { name: 'Wed', twitter: 1400, instagram: 1500 },
    { name: 'Thu', twitter: 1800, instagram: 1700 },
    { name: 'Fri', twitter: 2000, instagram: 1900 },
  ];

  return (
    <div style={{ maxWidth: 1000, margin:'2rem auto', fontFamily:'system-ui' }}>
      <h1>Social Media Dashboard</h1>
      <div style={{ display:'flex', gap:8, marginBottom: 16 }}>
        <select value={platform} onChange={e=>setPlatform(e.target.value)}>
          <option value="twitter">Twitter</option>
          <option value="instagram">Instagram</option>
        </select>
        <input value={content} onChange={e=>setContent(e.target.value)} style={{ flex:1 }} />
        <button onClick={schedule}>Schedule Post</button>
      </div>

      <div style={{ display:'grid', gridTemplateColumns:'1fr 1fr', gap:16 }}>
        <div style={{ padding:16, border:'1px solid #ddd', borderRadius:8 }}>
          <h3>Totals</h3>
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
        <div style={{ padding:16, border:'1px solid #ddd', borderRadius:8 }}>
          <h3>Impressions (sample)</h3>
          <ResponsiveContainer width="100%" height={300}>
            <LineChart data={chartData}>
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="name" />
              <YAxis />
              <Tooltip />
              <Legend />
              <Line type="monotone" dataKey="twitter" />
              <Line type="monotone" dataKey="instagram" />
            </LineChart>
          </ResponsiveContainer>
        </div>
      </div>
    </div>
  )
}





