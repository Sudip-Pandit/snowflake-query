# snowflake-query
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Snowflake Query Flow</title>
<link href="https://fonts.googleapis.com/css2?family=Oxanium:wght@400;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<style>
* { margin:0; padding:0; box-sizing:border-box; }

body {
  background: #060b14;
  font-family: 'Oxanium', sans-serif;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem 1rem;
  overflow-x: hidden;
}

/* animated star bg */
body::before {
  content:'';
  position:fixed; inset:0;
  background:
    radial-gradient(ellipse 80% 50% at 20% 20%, rgba(29,78,216,0.07) 0%, transparent 60%),
    radial-gradient(ellipse 60% 60% at 80% 80%, rgba(124,58,237,0.07) 0%, transparent 60%);
  pointer-events:none;
}

.canvas {
  width: 100%;
  max-width: 780px;
  position: relative;
}

/* ── Title ── */
.title-block {
  text-align: center;
  margin-bottom: 2rem;
  animation: fadeDown .7s ease both;
}
.title-eyebrow {
  font-family:'JetBrains Mono', monospace;
  font-size:.68rem;
  letter-spacing:.25em;
  color:#3b82f6;
  text-transform:uppercase;
  margin-bottom:.4rem;
}
.title-main {
  font-size: clamp(1.5rem, 4vw, 2.2rem);
  font-weight: 800;
  color: #fff;
  letter-spacing: -.02em;
  line-height: 1.1;
}
.title-main span { color: #29b6f6; }

.sql-pill {
  display: inline-flex;
  align-items: center;
  gap: .5rem;
  margin-top: .8rem;
  background: rgba(41,182,246,.07);
  border: 1px solid rgba(41,182,246,.2);
  border-radius: 6px;
  padding: .35rem .9rem;
  font-family: 'JetBrains Mono', monospace;
  font-size: .72rem;
  color: #7dd3fc;
}

/* ── Layer box ── */
.layer {
  border-radius: 16px;
  padding: 1.5rem 1.4rem 1.2rem;
  position: relative;
  animation: fadeUp .6s ease both;
}

.layer-cloud {
  background: linear-gradient(135deg, #0d1f3c 0%, #0a1628 100%);
  border: 1.5px solid #1e3a5f;
  box-shadow: 0 0 60px -20px rgba(41,182,246,.25), inset 0 1px 0 rgba(41,182,246,.1);
  animation-delay: .15s;
}

.layer-warehouse {
  background: linear-gradient(135deg, #1a0d30 0%, #120922 100%);
  border: 1.5px solid #3b1f6e;
  box-shadow: 0 0 60px -20px rgba(139,92,246,.25), inset 0 1px 0 rgba(139,92,246,.1);
  animation-delay: .35s;
}

/* Layer header badge */
.layer-header {
  display: flex;
  align-items: center;
  gap: .7rem;
  margin-bottom: 1.2rem;
  padding-bottom: .9rem;
  border-bottom: 1px solid rgba(255,255,255,.06);
}
.layer-icon {
  width: 36px; height: 36px;
  border-radius: 10px;
  display: flex; align-items: center; justify-content: center;
  font-size: 1.1rem;
  flex-shrink: 0;
}
.cloud-icon { background: rgba(41,182,246,.15); border: 1px solid rgba(41,182,246,.3); }
.wh-icon    { background: rgba(139,92,246,.15); border: 1px solid rgba(139,92,246,.3); }

.layer-title {
  font-size: .95rem;
  font-weight: 700;
  letter-spacing: .04em;
  text-transform: uppercase;
}
.cloud-title { color: #29b6f6; }
.wh-title    { color: #a78bfa; }

.layer-sub {
  font-family: 'JetBrains Mono', monospace;
  font-size: .62rem;
  color: #475569;
  margin-top:.1rem;
}

/* ── Steps grid ── */
.steps { display: flex; flex-direction: column; gap: .6rem; }

.step {
  display: flex;
  align-items: flex-start;
  gap: .85rem;
  padding: .75rem .9rem;
  border-radius: 10px;
  position: relative;
  transition: transform .2s, box-shadow .2s;
  cursor: default;
}
.step:hover { transform: translateX(4px); }

/* cloud steps */
.step-c { background: rgba(41,182,246,.05); border: 1px solid rgba(41,182,246,.1); }
.step-c:hover { box-shadow: -3px 0 0 #29b6f6, 0 4px 20px rgba(41,182,246,.1); }

/* warehouse steps */
.step-w { background: rgba(139,92,246,.05); border: 1px solid rgba(139,92,246,.1); }
.step-w:hover { box-shadow: -3px 0 0 #8b5cf6, 0 4px 20px rgba(139,92,246,.1); }

/* number badge */
.num {
  width: 28px; height: 28px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-family: 'JetBrains Mono', monospace;
  font-size: .65rem;
  font-weight: 700;
  flex-shrink: 0;
  margin-top: .05rem;
}
.num-c { background: rgba(41,182,246,.12); color: #29b6f6; border: 1.5px solid rgba(41,182,246,.3); }
.num-w { background: rgba(139,92,246,.12); color: #a78bfa; border: 1.5px solid rgba(139,92,246,.3); }

.step-body { flex: 1; min-width: 0; }

.step-title {
  font-size: .82rem;
  font-weight: 700;
  color: #e2e8f0;
  margin-bottom: .25rem;
}

.step-detail {
  font-family: 'JetBrains Mono', monospace;
  font-size: .65rem;
  color: #64748b;
  line-height: 1.6;
}
.step-detail .hi { color: #94a3b8; }
.step-detail .good { color: #34d399; }
.step-detail .warn { color: #fbbf24; }
.step-detail .info { color: #60a5fa; }

/* inline tag */
.badge {
  display: inline-flex;
  align-items: center;
  gap: .3rem;
  padding: .12rem .45rem;
  border-radius: 4px;
  font-family: 'JetBrains Mono', monospace;
  font-size: .58rem;
  font-weight: 600;
  margin-left: .4rem;
  vertical-align: middle;
}
.badge-green { background: rgba(52,211,153,.12); color: #34d399; border: 1px solid rgba(52,211,153,.2); }
.badge-red   { background: rgba(248,113,113,.12); color: #f87171; border: 1px solid rgba(248,113,113,.2); }
.badge-amber { background: rgba(251,191,36,.12);  color: #fbbf24; border: 1px solid rgba(251,191,36,.2); }
.badge-blue  { background: rgba(96,165,250,.12);  color: #60a5fa; border: 1px solid rgba(96,165,250,.2); }

/* stats row inside step */
.stats-row {
  display: flex;
  gap: .5rem;
  margin-top: .4rem;
  flex-wrap: wrap;
}
.stat {
  background: rgba(255,255,255,.04);
  border: 1px solid rgba(255,255,255,.07);
  border-radius: 5px;
  padding: .15rem .45rem;
  font-family: 'JetBrains Mono', monospace;
  font-size: .6rem;
}
.stat-val { font-weight: 700; }
.stat-lbl { color: #475569; margin-left: .2rem; }

/* pruning bar */
.prune-bar {
  margin-top: .5rem;
  height: 8px;
  border-radius: 99px;
  background: rgba(255,255,255,.05);
  overflow: hidden;
  position: relative;
}
.prune-fill-dead {
  position: absolute; left:0; top:0; bottom:0;
  width: 93.8%;
  background: linear-gradient(90deg, #f87171, #fb923c);
  border-radius: 99px 0 0 99px;
  animation: growBar .8s ease both;
  animation-delay: .8s;
}
.prune-fill-live {
  position: absolute; right:0; top:0; bottom:0;
  width: 6.2%;
  background: #34d399;
  border-radius: 0 99px 99px 0;
  animation: growBar .3s ease both;
  animation-delay: 1s;
}
.prune-labels {
  display: flex;
  justify-content: space-between;
  margin-top: .25rem;
  font-family: 'JetBrains Mono', monospace;
  font-size: .57rem;
}

/* node grid */
.node-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: .4rem;
  margin-top: .5rem;
}
.node-box {
  background: rgba(139,92,246,.08);
  border: 1px solid rgba(139,92,246,.2);
  border-radius: 7px;
  padding: .35rem .4rem;
  text-align: center;
  font-family: 'JetBrains Mono', monospace;
  font-size: .58rem;
}
.node-label { color: #a78bfa; font-weight: 700; display: block; }
.node-sub   { color: #475569; }

/* cache bar */
.cache-row {
  display: flex;
  gap: .35rem;
  margin-top: .45rem;
  align-items: center;
}
.cache-seg {
  height: 7px;
  border-radius: 4px;
}

/* ── Arrow between layers ── */
.inter-arrow {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
  margin: .6rem 0;
  animation: fadeUp .5s ease both;
  animation-delay: .25s;
}
.arrow-line {
  width: 2px;
  height: 28px;
  background: linear-gradient(to bottom, #1e3a5f, #3b1f6e);
}
.arrow-head {
  width: 0; height: 0;
  border-left: 7px solid transparent;
  border-right: 7px solid transparent;
  border-top: 10px solid #3b1f6e;
}
.arrow-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: .62rem;
  color: #334155;
  margin-top: .3rem;
  letter-spacing: .08em;
}

/* ── Footer ── */
.footer {
  text-align: center;
  margin-top: 1.6rem;
  animation: fadeUp .5s ease both;
  animation-delay: .5s;
}
.footer-text {
  font-family: 'JetBrains Mono', monospace;
  font-size: .6rem;
  color: #1e3a5f;
  letter-spacing: .12em;
  text-transform: uppercase;
}

/* step divider (within layer) */
.step-divider {
  height: 1px;
  background: rgba(255,255,255,.04);
  margin: .15rem 0;
}

/* ── Animations ── */
@keyframes fadeDown {
  from { opacity:0; transform:translateY(-14px); }
  to   { opacity:1; transform:translateY(0); }
}
@keyframes fadeUp {
  from { opacity:0; transform:translateY(14px); }
  to   { opacity:1; transform:translateY(0); }
}
@keyframes growBar {
  from { transform: scaleX(0); transform-origin: left; }
  to   { transform: scaleX(1); transform-origin: left; }
}

.step:nth-child(1) { animation: fadeUp .4s ease both; animation-delay:.2s; }
.step:nth-child(2) { animation: fadeUp .4s ease both; animation-delay:.28s; }
.step:nth-child(3) { animation: fadeUp .4s ease both; animation-delay:.36s; }
.step:nth-child(4) { animation: fadeUp .4s ease both; animation-delay:.44s; }
.step:nth-child(5) { animation: fadeUp .4s ease both; animation-delay:.52s; }
</style>
</head>
<body>
<div class="canvas">

  <!-- Title -->
  <div class="title-block">
    <div class="title-eyebrow">Snowflake Internals · End-to-End</div>
    <h1 class="title-main">What Happens When You<br>Run a <span>SQL Query?</span></h1>
    <div class="sql-pill">
      SELECT region, SUM(revenue) FROM sales
      WHERE date BETWEEN '2024-04-01' AND '2024-06-30'
      GROUP BY region
    </div>
  </div>

  <!-- ═══ CLOUD SERVICES LAYER ═══ -->
  <div class="layer layer-cloud">
    <div class="layer-header">
      <div class="layer-icon cloud-icon">☁️</div>
      <div>
        <div class="layer-title cloud-title">Cloud Services Layer</div>
        <div class="layer-sub">Orchestration · Metadata · Optimization · Zero compute cost</div>
      </div>
    </div>

    <div class="steps">

      <!-- Step 1 -->
      <div class="step step-c">
        <div class="num num-c">01</div>
        <div class="step-body">
          <div class="step-title">Client Submits SQL</div>
          <div class="step-detail">
            Query arrives via <span class="hi">JDBC / ODBC / HTTP</span> to Cloud Services.
            Session context, role <span class="hi">(ANALYST)</span>, and warehouse assignment established.
          </div>
        </div>
        <span class="badge badge-blue">Cloud Services</span>
      </div>

      <!-- Step 2 -->
      <div class="step step-c">
        <div class="num num-c">02</div>
        <div class="step-body">
          <div class="step-title">Parse & Validate</div>
          <div class="step-detail">
            SQL tokenized → <span class="hi">AST built</span>. Object names resolved (<span class="good">sales ✓</span>).
            RBAC permissions checked. Column types inferred (<span class="hi">revenue: NUMBER, date: DATE</span>).
          </div>
        </div>
        <span class="badge badge-blue">Query Parser</span>
      </div>

      <!-- Step 3 -->
      <div class="step step-c">
        <div class="num num-c">03</div>
        <div class="step-body">
          <div class="step-title">Cost-Based Optimization</div>
          <div class="step-detail">
            Best join order selected · <span class="info">Predicate pushed down</span> to scan level ·
            <span class="info">Column projection</span>: only <span class="hi">region, revenue, date</span> ·
            Partial aggregation at scan.
          </div>
        </div>
        <span class="badge badge-blue">Query Optimizer</span>
      </div>

      <!-- Step 4 -->
      <div class="step step-c">
        <div class="num num-c">04</div>
        <div class="step-body">
          <div class="step-title">Check Result Cache</div>
          <div class="step-detail">
            Identical query run ≤24h ago on unchanged data?
            <span class="badge badge-red">MISS — first run today</span>
            <br>On a <span class="good">cache HIT</span>: result returned instantly · zero compute · zero cost.
          </div>
        </div>
        <span class="badge badge-amber">Result Cache</span>
      </div>

      <!-- Step 5 -->
      <div class="step step-c">
        <div class="num num-c">05</div>
        <div class="step-body">
          <div class="step-title">Micro-Partition Pruning</div>
          <div class="step-detail">
            Metadata Store checks min/max <span class="hi">date</span> per partition — no data read needed.
          </div>
          <div class="prune-bar">
            <div class="prune-fill-dead"></div>
            <div class="prune-fill-live"></div>
          </div>
          <div class="prune-labels">
            <span style="color:#f87171">9,380 eliminated (93.8%)</span>
            <span style="color:#34d399">620 survive (6.2%)</span>
          </div>
          <div class="stats-row">
            <div class="stat"><span class="stat-val" style="color:#94a3b8">10,000</span><span class="stat-lbl">total</span></div>
            <div class="stat"><span class="stat-val" style="color:#f87171">9,380</span><span class="stat-lbl">pruned</span></div>
            <div class="stat"><span class="stat-val" style="color:#34d399">620</span><span class="stat-lbl">survive</span></div>
          </div>
        </div>
        <span class="badge badge-green">Pruning</span>
      </div>

    </div>
  </div>

  <!-- Arrow between layers -->
  <div class="inter-arrow">
    <div class="arrow-line"></div>
    <div class="arrow-head"></div>
    <div class="arrow-label">620 surviving partitions dispatched ↓</div>
  </div>

  <!-- ═══ VIRTUAL WAREHOUSE ═══ -->
  <div class="layer layer-warehouse">
    <div class="layer-header">
      <div class="layer-icon wh-icon">⚡</div>
      <div>
        <div class="layer-title wh-title">Virtual Warehouse · XL · 4 Nodes</div>
        <div class="layer-sub">MPP Compute · Shared-Nothing · Auto-Resume ~5s</div>
      </div>
    </div>

    <div class="steps">

      <!-- Step 6 -->
      <div class="step step-w">
        <div class="num num-w">06</div>
        <div class="step-body">
          <div class="step-title">Create Micro-Batches</div>
          <div class="step-detail">
            620 partitions packaged into ephemeral work units.
            <span class="hi">~155 micro-batches per node</span> · one batch per node per dispatch cycle.
          </div>
          <div class="node-grid" style="margin-top:.45rem">
            <div class="node-box"><span class="node-label">Node 1</span><span class="node-sub">~155 batches</span></div>
            <div class="node-box"><span class="node-label">Node 2</span><span class="node-sub">~155 batches</span></div>
            <div class="node-box"><span class="node-label">Node 3</span><span class="node-sub">~155 batches</span></div>
            <div class="node-box"><span class="node-label">Node 4</span><span class="node-sub">~155 batches</span></div>
          </div>
        </div>
        <span class="badge badge-blue">Dispatch</span>
      </div>

      <!-- Step 7 -->
      <div class="step step-w">
        <div class="num num-w">07</div>
        <div class="step-body">
          <div class="step-title">Warehouse Spin-Up & Dispatch</div>
          <div class="step-detail">
            Auto-resumes in <span class="good">~5 seconds</span>. Micro-batches distributed across all 4 nodes.
            Each node receives a <span class="hi">disjoint, independent set</span>.
            <span class="badge badge-amber">Credits start here</span>
          </div>
        </div>
        <span class="badge badge-blue">Virtual Warehouse</span>
      </div>

      <!-- Step 8 -->
      <div class="step step-w">
        <div class="num num-w">08</div>
        <div class="step-body">
          <div class="step-title">Read from Storage (or SSD Cache)</div>
          <div class="step-detail">Columnar read · only 3 of N columns fetched</div>
          <div class="cache-row">
            <div style="display:flex;align-items:center;gap:.3rem;flex:1">
              <div class="cache-seg" style="width:40%;background:linear-gradient(90deg,#34d399,#059669)"></div>
              <span style="font-family:'JetBrains Mono',monospace;font-size:.6rem;color:#34d399">40% Warm SSD — no remote I/O</span>
            </div>
          </div>
          <div class="cache-row">
            <div style="display:flex;align-items:center;gap:.3rem;flex:1">
              <div class="cache-seg" style="width:60%;background:linear-gradient(90deg,#3b82f6,#2563eb)"></div>
              <span style="font-family:'JetBrains Mono',monospace;font-size:.6rem;color:#60a5fa">60% Cold — S3/Blob fetch</span>
            </div>
          </div>
          <div class="step-detail" style="margin-top:.35rem">
            Columns read: <span class="hi">date · region · revenue</span> only
          </div>
        </div>
        <span class="badge badge-blue">Storage / Cache</span>
      </div>

      <!-- Step 9 -->
      <div class="step step-w">
        <div class="num num-w">09</div>
        <div class="step-body">
          <div class="step-title">MPP Parallel Processing</div>
          <div class="step-detail">
            Each node independently on its batches — <span class="hi">shared-nothing · no locks</span>
          </div>
          <div class="stats-row" style="margin-top:.4rem">
            <div class="stat"><span class="stat-val" style="color:#a78bfa">Filter</span><span class="stat-lbl">date BETWEEN</span></div>
            <div class="stat"><span class="stat-val" style="color:#a78bfa">Project</span><span class="stat-lbl">region, revenue</span></div>
            <div class="stat"><span class="stat-val" style="color:#a78bfa">Partial Agg</span><span class="stat-lbl">SUM COUNT AVG</span></div>
            <div class="stat"><span class="stat-val" style="color:#fbbf24">Shuffle</span><span class="stat-lbl">→ reduce</span></div>
            <div class="stat"><span class="stat-val" style="color:#34d399">ORDER BY</span><span class="stat-lbl">total_revenue DESC</span></div>
          </div>
        </div>
        <span class="badge badge-blue">MPP Compute</span>
      </div>

      <!-- Step 10 -->
      <div class="step step-w">
        <div class="num num-w">10</div>
        <div class="step-body">
          <div class="step-title">Return Results & Cache</div>
          <div class="step-detail">
            Final result flows <span class="hi">Cloud Services → Client</span>.
            Written to Result Cache with <span class="good">24h TTL</span>.
            Auto-suspend timer resets. <span class="good">Zero cost on next identical query.</span>
          </div>
        </div>
        <span class="badge badge-green">Client Response</span>
      </div>

    </div>
  </div>

  <!-- Footer -->
  <div class="footer">
    <div class="footer-text">Snowflake · End-to-End Query Flow · 10-Stage Pipeline</div>
  </div>

</div>
</body>
</html>
