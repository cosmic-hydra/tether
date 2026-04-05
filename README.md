

# Tether: Decentralized P2P Compute Fabric

Tether is a Decentralized Physical Infrastructure Network (DePIN) designed to coordinate distributed, high-performance computing tasks. The system facilitates an asynchronous, two-sided marketplace where a Control Node (Orchestrator) manages job distribution, and Worker Nodes (Contributors) provide computational power through secure, virtualized environments. 

Targeting the $2.3B render farm market and the 10M+ global Blender user base, Tether's core innovation is capturing "spare headroom"—utilizing partial GPU/CPU resources from actively used machines rather than waiting for machines to go completely idle.

## Current State: Phase 1 Implementation

The codebase currently hosted in this repository represents **Phase 1: Node Agent & Basic Orchestrator**. 

It is a fully functional, minimal viable product (MVP) capable of distributed rendering:
* **The Orchestrator:** A FastAPI backend that manages job queues, maps frame ranges for `.blend` files, and aggregates results via secure ngrok tunneling.
* **The Node Agent:** A Python-based client that establishes a handshake, downloads assets, and triggers a sandboxed Docker container (Linux/Blender) to execute the render without requiring local host installations.
* **Security:** A zero-footprint environment variable strategy ensures that proprietary node paths and Orchestrator API keys are never exposed.

## The Total Blueprint: Network Architecture

As the network matures, Tether will operate across three distinct modes:
1. **Global Network:** Spare compute harvested from any connected machine worldwide. Target: Indie artists, AI hobbyists.
2. **LAN Cluster Mode:** A self-hosted Orchestrator running on a studio's local network, turning existing artist workstations into a private render farm with zero new hardware investment.
3. **Overflow:** When a LAN Cluster is saturated during crunch periods, encrypted jobs automatically spill over to the Global Network.

### The Node Agent (Supply Side)
The Node Agent is the foundation of the network. Future iterations will include:
* **Dynamic Headroom Detection:** Continuous telemetry sampling (via NVML for NVIDIA, psutil for CPU/RAM). The agent will seamlessly throttle its resource usage based on the host user's active workload.
* **Strict Sandboxing:** No host filesystem access, network namespace isolation, resource hard caps, and kernel-level Seccomp profiles to prevent container escapes.

### Smart Allocation System
Naive dispatch (matching any node that meets minimum specs) fails in real-world decentralized systems. Tether's routing engine will score candidate nodes across six dimensions before assignment:
* **Resource Fit:** Hard filter for VRAM/CPU/RAM minimums.
* **Specialization:** Preferential routing to nodes that already have required assets/models cached.
* **Reliability:** Scored based on historical job completion rates.
* **Stability:** Variance of available resources over the last 60 minutes.
* **Proximity:** Transfer speed latency to the storage endpoint.
* **Price:** Buyer-configurable weight for budget optimization.

### Verification Matrix (Anti-Cheat)
Operating a trustless compute network requires making cheating economically irrational.
1. **Redundant Execution:** For deterministic jobs (like Blender), chunks are sent to multiple nodes. Hashes of the outputs are compared. Matching nodes are paid; outliers are slashed.
2. **Reputation Scaling:** As nodes successfully complete verified work, their redundancy requirements drop (from 100% to 2%), lowering costs for the network.
3. **Hardware Attestation (TEE):** For highly confidential studio overflow, jobs run inside Intel SGX or AMD SEV enclaves, ensuring code integrity and data privacy.
4. **Optimistic Verification:** For non-deterministic jobs (like AI inference), random verifiers challenge statistical samples of submitted execution traces.

### Two-Token Economic Model
To protect buyers from crypto-market volatility while adequately incentivizing supply:
* **Fiat/USDC (Payment Layer):** Buyers pay for compute using standard credit cards or stablecoins. Pricing is predictable and transparent.
* **$DPIN (Utility Layer):** Node operators must stake the protocol token to accept jobs. This acts as a security deposit against fraudulent work. High stakes unlock premium job tiers, and the token handles protocol governance.

---

## Quick Start (Phase 1 Codebase)

### Prerequisites
* **Orchestrator:** Python 3.10+, ngrok account.
* **Node Agent:** Python 3.10+, Docker Desktop installed and running.

### 1. Orchestrator Setup (Master)
1. Clone the repository and install dependencies:
   ```bash
   pip install fastapi uvicorn ngrok python-dotenv
   ```
2. Create a `.env` file and add your ngrok credentials:
   ```text
   NGROK_AUTH_TOKEN=your_token_here
   ```
3. Place your target `.blend` file into the `/jobs` directory.
4. Launch the server:
   ```bash
   python orchestrator.py
   ```
   *Note the public ngrok URL generated in the console.*

### 2. Node Agent Setup (Worker)
1. Install dependencies:
   ```bash
   pip install requests python-dotenv
   ```
2. Copy the `.env.example` file to `.env` and configure the Orchestrator URL:
   ```text
   ORCHESTRATOR_URL=https://your-provided-url.ngrok-free.app
   ```
3. Start the compute node:
   ```bash
   python node_agent.py
   ```

---

## Build Phases & Roadmap

* **Phase 1 (Current):** Node Agent MVP + Basic Orchestrator + Hot Storage Integration.
* **Phase 2:** Smart Allocation Engine + End-to-End Blender Plugin integration + Redundant Execution verification.
* **Phase 3:** On-Chain Layer deployment (Staking contracts, Escrow, $DPIN token logic).
* **Phase 4:** LAN Cluster Mode release + Initial Studio Onboarding + Encrypted Overflow.
* **Phase 5:** Fiat On-Ramp (Stripe integration) + Comprehensive Contributor Dashboard.
* **Phase 6:** Cold Distributed Storage network (Proof of Storage verification).
* **Phase 7:** Workload Expansion (AI Inference model caching + FFmpeg Video Transcoding).
* **Phase 8:** Enterprise Readiness (TEE Hardware Attestation + SOC 2 compliance).
