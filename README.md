# 🚀 Monitor Express JS App using Prometheus & Grafana

A complete beginner-friendly guide to monitor your **Express.js application** using **Prometheus + Grafana**.

This tutorial covers:

* 📊 Metrics collection
* ⚡ Prometheus integration
* 📈 Grafana visualization

---

## 📌 Prerequisites

Make sure you have:

* Node.js & npm installed
* Prometheus installed and running
* Grafana installed and running

---

## ⚡ Step 1: Initialize Node.js Project

```bash
mkdir express-prometheus-grafana
cd express-prometheus-grafana
npm init -y
npm install express prom-client
```

---

## ⚡ Step 2: Create Express Server with Metrics

Create a file `server.js` and add:

```javascript
const express = require("express");
const client = require("prom-client");

const app = express();
const PORT = 3001;

// Register metrics
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metric
const httpRequestDuration = new client.Histogram({
  name: "http_request_duration_seconds",
  help: "Duration of HTTP requests in seconds",
  labelNames: ["method", "route", "status_code"],
});
register.registerMetric(httpRequestDuration);

// Middleware
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on("finish", () => {
    end({ method: req.method, route: req.path, status_code: res.statusCode });
  });
  next();
});

// Routes
app.get("/", (req, res) => {
  res.send("Hello, Express with Prometheus!");
});

// Metrics endpoint
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});

// Start server
app.listen(PORT, "0.0.0.0", () => {
  console.log(`✅ Server running on http://localhost:${PORT}`);
});
```

---

### ▶️ Run the Server

```bash
node server.js
```

👉 Open in browser:
`http://localhost:3001/metrics`

---

## ⚡ Step 3: Configure Prometheus

Edit config file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add:

```yaml
scrape_configs:
  - job_name: "express-app"
    static_configs:
      - targets: ["<your-system-ip>:3001"]
```

👉 Replace `<your-system-ip>` with your IP
(e.g., `192.168.1.100:3001`)

---

### 🔄 Restart Prometheus

```bash
sudo systemctl restart prometheus
```

👉 Verify targets:
`http://localhost:9090/classic/targets`

Status should be **UP ✅**

---

## ⚡ Step 4: Visualize in Grafana

* Open: `http://localhost:3000`
* Go to **Dashboards → New Dashboard → Add Query**
* Select **Prometheus**

### 🔹 Query:

```bash
sum by (instance, method) (rate(http_request_duration_seconds_count[5m]))
```

### 🔹 Settings:

* **Legend:** `{{instance}} - {{method}}`
* **Unit:** requests/sec (rps)

---

## 📊 Example Dashboard

* HTTP Requests per second (RPS)
* Average Request Duration
* Success vs Failed Requests

---

## 🚀 Conclusion

You have successfully:

* Built an Express.js app with metrics
* Integrated Prometheus
* Visualized data in Grafana

🎯 Your app is now fully monitored!

---

## 🌟 Author

Made with Saurav❤️

---

⭐ If you found this useful, don’t forget to **star the repo**!

**Happy Monitoring 🚀**
