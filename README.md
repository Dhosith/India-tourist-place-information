# India-tourist-place-information
// Full-stack Health Tracker App - Node.js Backend (Express)
// File: server/index.js

const express = require("express");
const cors = require("cors");
const bodyParser = require("body-parser");
const { CosmosClient } = require("@azure/cosmos");
require("dotenv\config");

const app = express();
const port = process.env.PORT || 4000;

app.use(cors());
app.use(bodyParser.json());

// Cosmos DB setup
const cosmosClient = new CosmosClient({
  endpoint: process.env.COSMOS_DB_ENDPOINT,
  key: process.env.COSMOS_DB_KEY
});

const database = cosmosClient.database("HealthTrackerDB");
const container = database.container("UserHealthData");

// Routes
app.get("/api/user/:id/data", async (req, res) => {
  const userId = req.params.id;
  const query = {
    query: "SELECT * FROM c WHERE c.userId = @userId",
    parameters: [{ name: "@userId", value: userId }]
  };

  try {
    const { resources } = await container.items.query(query).fetchAll();
    res.json(resources);
  } catch (err) {
    console.error(err);
    res.status(500).send("Error fetching user data");
  }
});

app.post("/api/user/:id/data", async (req, res) => {
  const userId = req.params.id;
  const healthEntry = { ...req.body, userId, id: `${userId}-${Date.now()}` };

  try {
    const { resource } = await container.items.create(healthEntry);
    res.status(201).json(resource);
  } catch (err) {
    console.error(err);
    res.status(500).send("Error saving health data");
  }
});

app.post("/api/ai/suggestions", async (req, res) => {
  const { healthData } = req.body;

  // Simulated AI suggestion logic (replace with Azure OpenAI call if desired)
  const suggestions = [];
  if (healthData.sleepHours < 7) suggestions.push("Try to get at least 7 hours of sleep.");
  if (healthData.steps < 10000) suggestions.push("Aim for 10,000 steps per day.");
  if (healthData.heartRate > 80) suggestions.push("Your heart rate is a bit high; consider relaxation techniques.");

  res.json({ suggestions });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
