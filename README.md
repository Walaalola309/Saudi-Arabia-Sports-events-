# Saudi-Arabia-Sports-events-// Full-Stack Code for Saudi Sports Events Website

/** Backend: server.js **/
const express = require("express");
const axios = require("axios");
const cors = require("cors");
require("dotenv").config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(express.json());

// Mock Events Endpoint
app.get("/api/events", (req, res) => {
  const events = [
    { name: "Formula 1 Saudi GP", date: "2024-12-15", venue: "Jeddah Corniche Circuit" },
    { name: "Football Match", date: "2024-11-30", venue: "King Fahd Stadium, Riyadh" },
    { name: "E-Sports Tournament", date: "2024-12-10", venue: "Riyadh Front" },
  ];
  res.json(events);
});

// Nearby Places Endpoint
app.get("/api/nearby", async (req, res) => {
  const { lat, lng } = req.query;
  const apiKey = process.env.GOOGLE_MAPS_API_KEY;
  const url = `https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=${lat},${lng}&radius=2000&type=point_of_interest&key=${apiKey}`;

  try {
    const response = await axios.get(url);
    res.json(response.data.results);
  } catch (error) {
    console.error(error);
    res.status(500).send("Error fetching nearby places");
  }
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});


/** Frontend: React Components **/
// App.js
import React from "react";
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";
import Homepage from "./components/Homepage";
import EventDetails from "./components/EventDetails";
import NearbyPlaces from "./components/NearbyPlaces";

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Homepage />} />
        <Route path="/events/:id" element={<EventDetails />} />
        <Route path="/nearby" element={<NearbyPlaces />} />
      </Routes>
    </Router>
  );
}

export default App;


// Homepage.js
import React, { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import axios from "axios";

function Homepage() {
  const [events, setEvents] = useState([]);

  useEffect(() => {
    axios.get("http://localhost:5000/api/events").then((response) => {
      setEvents(response.data);
    });
  }, []);

  return (
    <div>
      <h1>Upcoming Sports Events</h1>
      <ul>
        {events.map((event, index) => (
          <li key={index}>
            <Link to={`/events/${index}`}>{event.name}</Link> - {event.date}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default Homepage;


// EventDetails.js
import React from "react";
import { useParams } from "react-router-dom";

function EventDetails() {
  const { id } = useParams();

  return (
    <div>
      <h2>Event Details</h2>
      <p>Details for event ID: {id}</p>
      {/* Fetch and display specific event details here */}
    </div>
  );
}

export default EventDetails;


// NearbyPlaces.js
import React, { useState } from "react";
import axios from "axios";

function NearbyPlaces() {
  const [places, setPlaces] = useState([]);
  const [location, setLocation] = useState({ lat: "", lng: "" });

  const fetchPlaces = () => {
    axios
      .get("http://localhost:5000/api/nearby", {
        params: {
          lat: location.lat,
          lng: location.lng,
        },
      })
      .then((response) => {
        setPlaces(response.data);
      });
  };

  return (
    <div>
      <h2>Nearby Places</h2>
      <input
        type="text"
        placeholder="Latitude"
        value={location.lat}
        onChange={(e) => setLocation({ ...location, lat: e.target.value })}
      />
      <input
        type="text"
        placeholder="Longitude"
        value={location.lng}
        onChange={(e) => setLocation({ ...location, lng: e.target.value })}
      />
      <button onClick={fetchPlaces}>Search Nearby Places</button>

      <ul>
        {places.map((place, index) => (
          <li key={index}>{place.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default NearbyPlaces;


/** Deployment **/
// 1. Push the frontend and backend folders to a GitHub repository.
// 2. Deploy the backend using Heroku or AWS (include .env for sensitive data).
// 3. Deploy the frontend using Netlify or Vercel.

// Final Website: Combines event details, nearby places, and dynamic updates.
