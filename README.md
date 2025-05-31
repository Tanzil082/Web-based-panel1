# Web-based-panel1
// Project: Web-Based Admin Panel with Web Scraping

// --- Backend (Node.js + Express + Cheerio for scraping) ---
// server/index.js
const express = require('express');
const axios = require('axios');
const cheerio = require('cheerio');
const cors = require('cors');
const mongoose = require('mongoose');

const app = express();
app.use(cors());
app.use(express.json());

// Connect to MongoDB Atlas
mongoose.connect('YOUR_MONGODB_ATLAS_URI', { useNewUrlParser: true, useUnifiedTopology: true });
const Book = mongoose.model('Book', new mongoose.Schema({ title: String, price: String, rating: String }));

// Scrape Route
app.get('/scrape', async (req, res) => {
  const response = await axios.get('https://books.toscrape.com');
  const $ = cheerio.load(response.data);
  const books = [];

  $('.product_pod').each((i, el) => {
    const title = $(el).find('h3 a').attr('title');
    const price = $(el).find('.price_color').text();
    const rating = $(el).find('.star-rating').attr('class').split(' ')[1];
    books.push({ title, price, rating });
  });

  await Book.deleteMany();
  await Book.insertMany(books);
  res.json({ message: 'Scraping complete', count: books.length });
});

// Get All Books
app.get('/books', async (req, res) => {
  const books = await Book.find();
  res.json(books);
});

app.listen(5000, () => console.log('Server running on port 5000'));

// --- Frontend (React + Bootstrap) ---
// client/src/App.js
import React, { useEffect, useState } from 'react';
import axios from 'axios';
import './App.css';

function App() {
  const [books, setBooks] = useState([]);

  const fetchBooks = async () => {
    const res = await axios.get('http://localhost:5000/books');
    setBooks(res.data);
  };

  const handleScrape = async () => {
    await axios.get('http://localhost:5000/scrape');
    fetchBooks();
  };

  useEffect(() => {
    fetchBooks();
  }, []);

  return (
    <div className="container">
      <h1 className="mt-4">Admin Panel - Book Data</h1>
      <button className="btn btn-primary mb-3" onClick={handleScrape}>Scrape Now</button>
      <table className="table table-bordered">
        <thead>
          <tr>
            <th>Title</th>
            <th>Price</th>
            <th>Rating</th>
          </tr>
        </thead>
        <tbody>
          {books.map((book, index) => (
            <tr key={index}>
              <td>{book.title}</td>
              <td>{book.price}</td>
              <td>{book.rating}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default App;


