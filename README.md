# fullstack10.1
/**
 * ================================
 * 🌐 FULL STACK CRUD DEMO (ALL-IN-ONE)
 * Backend: Node.js + Express + MongoDB (Mongoose)
 * Frontend Simulation: Basic CLI Inputs using readline (Simulates React/Axios)
 * ================================
 */

const express = require("express");
const mongoose = require("mongoose");
const bodyParser = require("body-parser");
const readline = require("readline");
const axios = require("axios");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(bodyParser.json());

/* =======================
   DATABASE CONNECTION
======================= */
mongoose
  .connect("mongodb://127.0.0.1:27017/all_in_one_crud", {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => console.log("✅ MongoDB Connected Successfully"))
  .catch((err) => console.log("❌ MongoDB Error:", err));

/* =======================
   MONGOOSE SCHEMA
======================= */
const productSchema = new mongoose.Schema({
  name: String,
  price: Number,
  quantity: Number,
});

const Product = mongoose.model("Product", productSchema);

/* =======================
   CRUD API ENDPOINTS
======================= */

// ➕ CREATE
app.post("/api/products", async (req, res) => {
  try {
    const product = new Product(req.body);
    await product.save();
    res.json(product);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// 📖 READ
app.get("/api/products", async (req, res) => {
  const products = await Product.find();
  res.json(products);
});

// ✏️ UPDATE
app.put("/api/products/:id", async (req, res) => {
  try {
    const updated = await Product.findByIdAndUpdate(req.params.id, req.body, {
      new: true,
    });
    res.json(updated);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// ❌ DELETE
app.delete("/api/products/:id", async (req, res) => {
  try {
    await Product.findByIdAndDelete(req.params.id);
    res.json({ message: "Product deleted successfully" });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

/* =======================
   START EXPRESS SERVER
======================= */
const PORT = 5000;
app.listen(PORT, () =>
  console.log(`🚀 Server running at http://localhost:${PORT}`)
);

/* =======================
   CLI FRONTEND SIMULATION
======================= */
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

console.log(`
🛒 WELCOME TO FULL STACK CRUD (Simulated Frontend)
1️⃣ Add Product
2️⃣ View Products
3️⃣ Update Product
4️⃣ Delete Product
5️⃣ Exit
`);

const BASE_URL = `http://localhost:${PORT}/api/products`;

const ask = () => {
  rl.question("Enter choice (1-5): ", async (choice) => {
    switch (choice) {
      case "1":
        rl.question("Enter product name: ", (name) => {
          rl.question("Enter price: ", (price) => {
            rl.question("Enter quantity: ", async (quantity) => {
              await axios.post(BASE_URL, { name, price, quantity });
              console.log("✅ Product added successfully!");
              ask();
            });
          });
        });
        break;

      case "2":
        const res = await axios.get(BASE_URL);
        console.table(res.data);
        ask();
        break;

      case "3":
        const list = await axios.get(BASE_URL);
        console.table(list.data);
        rl.question("Enter ID to update: ", (id) => {
          rl.question("New name: ", (name) => {
            rl.question("New price: ", (price) => {
              rl.question("New quantity: ", async (quantity) => {
                await axios.put(`${BASE_URL}/${id}`, { name, price, quantity });
                console.log("✏️ Product updated!");
                ask();
              });
            });
          });
        });
        break;

      case "4":
        const items = await axios.get(BASE_URL);
        console.table(items.data);
        rl.question("Enter ID to delete: ", async (id) => {
          await axios.delete(`${BASE_URL}/${id}`);
          console.log("🗑️ Product deleted!");
          ask();
        });
        break;

      case "5":
        console.log("👋 Exiting...");
        rl.close();
        process.exit(0);

      default:
        console.log("❗ Invalid choice, try again.");
        ask();
    }
  });
};

setTimeout(() => ask(), 1500);
