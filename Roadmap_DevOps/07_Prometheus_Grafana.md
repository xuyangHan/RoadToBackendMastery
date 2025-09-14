# Exposing Metrics the Right Way: Prometheus + Grafana in Action

## **Introduction**

In [my previous blog](../Roadmap_DevOps/06_Application_Performance_Management.md), we looked at Application Performance Management (APM) from a DevOps perspective—how logs and traces can help teams understand what’s happening inside their systems. But monitoring doesn’t stop there. The next step is to move beyond ad-hoc inspection and start exposing **metrics** in a structured, consistent way.

Metrics are the heartbeat of any modern application. They provide measurable signals about system health, reliability, and performance trends. With the right metrics in place, teams can answer questions like:

- *Is our service handling more requests than usual?*
- *Are error rates creeping up?*
- *How does today compare to last week?*

Without metrics, you’re left guessing or constantly pulling raw logs just to understand whether the system is behaving normally.

### **Why Prometheus Beats APIs for Metrics**

Let’s be clear: APIs aren’t useless in monitoring. They’re great for drilldowns or fetching detailed records. But as the main vehicle for metrics, they fall short.

- **APIs only give snapshots**: you make a request, get the current value, and that’s it. If you want history, you’re responsible for polling, storing, and aggregating.
- **Custom dashboards are expensive**: with APIs, every team ends up building its own query engine, storage, and visualization—work that doesn’t directly add value to the product.

Prometheus flips this model. Instead of pulling data manually, Prometheus **scrapes endpoints automatically** on a schedule. Each scrape records the metric values and builds them into a **time-series database**. That means you get history “for free.”

On top of that:

- Prometheus offers **PromQL**, a powerful query language that makes aggregations, comparisons, and trend analysis straightforward.
- Metrics become **cheap to expose**: the application only needs to provide its current values at `/metrics`, and Prometheus takes care of the rest.
- Grafana integrates natively with Prometheus, giving you dashboards, panels, and sharing options with no extra effort.

So the right way to think about it is this: APIs are still useful when you need detailed drilldowns, but **metrics endpoints should be designed specifically for Prometheus**. That way, you maximize the strengths of both approaches without duplicating effort.

That’s where **Prometheus + Grafana** comes in. Together, they offer a simple, industry-standard way to expose, collect, and visualize metrics without reinventing the wheel. Instead of manually calling APIs and stitching together dashboards, you let Prometheus handle the heavy lifting, and Grafana gives you the rich visualization layer out of the box.

---

## **Designing Metrics in a C# Project**

When you expose metrics from an ASP.NET Core app, you’re essentially giving Prometheus a **window into your application’s state**. That means you need to decide:

- **What to measure**
- **How to represent it** (counter, gauge, histogram, etc.)
- **Which labels to attach** (to make aggregation meaningful, without exploding into high-cardinality chaos).

Let’s walk through a practical example: imagine you’re building the Uniqlo mobile app and you want to track **how many users log in**.

### **1. Choosing the Right Metric Type**

- A **counter** is the natural fit here: every time a user logs in, the counter increases. Counters never decrease, which makes them ideal for tracking events over time.
- You might also want a **gauge** for the number of currently active sessions — something that goes up when users log in, and down when they log out.

### **2. Designing Labels**

Labels let you slice the data in Prometheus queries later. For logins, useful labels might be:

- `region` → to compare user activity between, say, North America and Asia.
- `platform` → to see logins from iOS vs. Android.

What you should *not* do is label metrics with **high-cardinality identifiers** like `user_id` or `session_uid`. Prometheus stores a separate time-series for every unique label combination — meaning if you have millions of users, your monitoring system will melt down.

So, a good metric might look like this:

``` plaintext
app_user_logins_total{region="NorthAmerica", platform="ios"} 124
app_user_logins_total{region="NorthAmerica", platform="android"} 203
app_user_logins_total{region="Asia", platform="ios"} 89
```

### **3. Setting Up in ASP.NET Core**

Using the [`prometheus-net`](https://github.com/prometheus-net/prometheus-net) library, you can wire this up quickly.

```csharp
using Prometheus;

public class MetricsCollector
{
    private static readonly Counter UserLogins = Metrics
        .CreateCounter("app_user_logins_total", "Total number of user logins",
            new CounterConfiguration
            {
                LabelNames = new[] { "region", "platform" }
            });

    private static readonly Gauge ActiveSessions = Metrics
        .CreateGauge("app_active_sessions", "Number of currently active user sessions",
            new GaugeConfiguration
            {
                LabelNames = new[] { "region", "platform" }
            });

    public static void RecordLogin(string region, string platform)
    {
        UserLogins.WithLabels(region, platform).Inc();
        ActiveSessions.WithLabels(region, platform).Inc();
    }

    public static void RecordLogout(string region, string platform)
    {
        ActiveSessions.WithLabels(region, platform).Dec();
    }
}
```

Then in `Program.cs` (or `Startup.cs` depending on your project):

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseRouting();

// Expose default and custom metrics at /metrics
app.UseHttpMetrics();
app.UseMetricServer();

app.MapGet("/login/{region}/{platform}", (string region, string platform) =>
{
    MetricsCollector.RecordLogin(region, platform);
    return Results.Ok($"Login recorded for {region}/{platform}");
});

app.MapGet("/logout/{region}/{platform}", (string region, string platform) =>
{
    MetricsCollector.RecordLogout(region, platform);
    return Results.Ok($"Logout recorded for {region}/{platform}");
});

app.Run();
```

### **4. Why This Matters**

By designing metrics this way:

- **Prometheus** handles scraping and building the time-series automatically.
- **Grafana** can visualize logins by region or platform with just a few clicks.
- You avoid the pitfalls of API snapshots and get a continuous history of user behavior.

---

## **Prometheus Setup**

Now that our ASP.NET Core app is exposing metrics at `/metrics`, the next step is to get Prometheus to scrape them.

Prometheus itself is just a **time-series database + scraper**. Your app only reports **current values** (like “124 logins right now”). Prometheus takes care of querying that endpoint on a schedule, appending a timestamp, and building a historical view automatically.

First, you’ll need Prometheus itself. You can download the latest release directly from the [official Prometheus website](https://prometheus.io/download/). For local testing, grab the package for your operating system, unzip it, and you’ll find a `prometheus` (or `prometheus.exe` on Windows) binary inside. Running it is as simple as pointing to your config file:

```bash
./prometheus --config.file=prometheus.yml
```

This will start Prometheus with a local web UI at [http://localhost:9090](http://localhost:9090), where you can explore metrics and test queries. Running it on a server is a bit trickier (you’d typically set it up as a service or container), but the workflow is essentially the same: place your `prometheus.yml` in the right spot and run the binary with that config.

### Example `prometheus.yml`

Here’s a minimal configuration to scrape your app:

```yaml
global:
  scrape_interval: 15s    # how often Prometheus scrapes all targets
  evaluation_interval: 15s

scrape_configs:
  - job_name: "uniqlo-app-metrics"
    metrics_path: "/metrics"   # endpoint in your ASP.NET Core app
    static_configs:
      - targets: ["localhost:7301"]
        labels:
          app: "uniqlo-store"
          env: "dev"
```

### Breaking it down

- **`scrape_interval`**: Every 15 seconds, Prometheus will call your `/metrics` endpoint.
- **`job_name`**: A logical group name for this target (you’ll use it later in queries).
- **`metrics_path`**: By default Prometheus looks at `/metrics`, but you can change it if needed.
- **`targets`**: The host\:port where your app runs. In real deployments, this might be Kubernetes service DNS names or ECS tasks.
- **`labels`**: Extra metadata you want attached to *every metric* scraped from this target (e.g., `env="prod"`, `app="mobile-api"`).

With this in place, Prometheus starts polling your app. Each scrape produces entries like:

``` plaintext
app_user_logins_total{region="NorthAmerica", platform="ios", job="uniqlo-app-metrics", env="dev"} 124
```

Notice how Prometheus automatically adds `job` and any custom labels you configured in `prometheus.yml`. This metadata is what makes filtering and aggregating later in Grafana so powerful.

### Best Practice: Don’t Query the DB on Scrape

A crucial point: your `/metrics` endpoint should **not** hit the database every time Prometheus scrapes. Since scraping happens often (every 15s in the example), doing DB queries on each request will overload your system.

Instead, the industry standard is:

1. Collect data periodically in the background (e.g., poll DB every 1–5 minutes).
2. Cache or pre-aggregate the values in memory (or into a dedicated “metrics table”).
3. When Prometheus scrapes `/metrics`, simply return the cached values.

This pattern keeps your endpoint **fast, cheap, and reliable**, while Prometheus handles the heavy lifting of historical storage and visualization.

---

## **Grafana Visualization**

Prometheus on its own is great at scraping and storing metrics, but the real power comes when you can visualize those numbers over time. That’s where [Grafana](https://grafana.com/) comes in. Grafana is the industry-standard visualization tool that integrates seamlessly with Prometheus and lets you create powerful dashboards.

### Setting up Grafana locally

For testing, you can download Grafana from the [official download page](https://grafana.com/grafana/download). On Windows and macOS, it comes as an installer (`.msi` or `.dmg`). On Linux, you can grab a `.tar.gz` archive or install it with your package manager. After installation, start the Grafana server, and by default you can reach it at [http://localhost:3000](http://localhost:3000).

The default login is:

- **Username:** `admin`
- **Password:** `admin` (you’ll be asked to change it on first login).

### Connecting Prometheus as a Data Source

1. Go to **Configuration → Data Sources** in Grafana.
2. Choose **Prometheus** from the list of built-in integrations.
3. Enter the Prometheus server URL, e.g., `http://localhost:9090`.
4. Save & Test — Grafana will confirm the connection.

Now Grafana can query all the metrics that Prometheus has been scraping.

### Building Dashboards and Panels

Grafana organizes visualizations into **dashboards**, each made up of one or more **panels**. A panel could be a time-series chart, gauge, bar chart, or even a single stat number.

For example, with our earlier `app_user_logins_total` metric, you could create:

- A **time-series panel** showing logins broken down by region.
- A **stat panel** displaying the current number of iOS vs Android users.
- A **bar panel** comparing login counts across regions at a glance.

Because metrics carry labels like `region` and `platform`, you can aggregate or filter them with simple PromQL queries such as:

```promql
sum by (region) (app_user_logins_total)
```

This query would show the total logins per region across all platforms.

### Embedding Panels with Real-Time Updates

Once your dashboard is ready, Grafana makes sharing easy. Each panel has a **Share** option where you can copy an `<iframe>` snippet. You can embed that iframe in internal portals, documentation wikis, or even team dashboards. The panel will stay connected to Grafana and **update in real time** as new metrics flow in.

This way, instead of static reports or screenshots, you give your team live, interactive views into system health.

---

## **Lessons Learned & Conclusion**

Through this exercise, a few key lessons stand out:

- **Keep metrics lightweight and aggregated** — avoid exposing raw event streams; metrics should remain predictable and cheap to scrape.
- **Separate concerns** — business metrics (like logins, transactions per second) should be distinct from system metrics (CPU, memory, request latency). This keeps dashboards more meaningful and prevents signal loss in the noise.
- **Metrics endpoints are not APIs** — treat them as structured, query-friendly data sources optimized for monitoring, not as sources of detailed transactional data. When needed, use your APIs or logs for deeper contextual drilldowns.

With these principles in mind, Prometheus and Grafana together provide a standardized, scalable, and low-friction way to monitor applications. Prometheus ensures reliable collection and time-series storage, while Grafana unlocks the power of visualization, real-time dashboards, and easy embedding into your internal portals or wikis.

This foundation is just the beginning: once you have metrics flowing, you can extend the ecosystem with **Prometheus alerting rules**, **Grafana alerts**, or even integration into **CI/CD pipelines** for automated quality gates. In other words, metrics aren’t just for monitoring—they can actively shape how your teams build, ship, and operate software at scale.
