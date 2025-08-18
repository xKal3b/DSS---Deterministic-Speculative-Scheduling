# DSS---Deterministic-Speculative-Scheduling
A runtime execution model that allows blockchain transactions to be executed in parallel speculatively, while ensuring a deterministic final order. It reduces lock contention, adapts throughput dynamically to network conditions, and improves stability under high load without sacrificing consensus safety.
# Deterministic Speculative Scheduling (DSS)

Deterministic Speculative Scheduling (DSS) is a new runtime model for Solana-style blockchains. It improves throughput and stability by executing transactions speculatively in parallel, while committing results in a deterministic final order.  

### 🚀 Why DSS?
- **+15–20% throughput** under stress (NFT mints, liquidation cascades).  
- **Lower latency variance** → smoother p95/p99.  
- **Adaptive scaling** based on hardware + network conditions.  

### 📌 Key Features
- Speculative parallel execution.  
- Deterministic final ordering.  
- Adaptive execution window sizing.  

### 📬 Contact
Author: **Kaleb Barnhart** X (@xkal3b)  
Email: **kbrnhrt@gmail.com**  

---

⚠️ **Disclaimer:** DSS is experimental. Results are based on synthetic simulations; real-world validator performance may vary.
