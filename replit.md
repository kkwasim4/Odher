# OdherApp Explorer - Enterprise Multi-Chain Token Intelligence Platform

## Overview
OdherApp Explorer is an enterprise-grade on-chain intelligence platform designed to simplify blockchain token analysis with live blockchain data across all core features. It aggregates real-time data from 23+ EVM networks and Solana, providing instant verification, live market pricing, liquidity analysis, comprehensive risk assessment, and AI-powered summaries. Its core mission is to be the trusted source for deep-dive token contract analysis, enhancing transparency and user safety in the crypto space.

## User Preferences
- **100% live blockchain data** - All features use authentic on-chain information from RPC endpoints
- Focus on visual excellence and data accuracy
- Professional, crypto-industry standard aesthetics
- Fast analysis (< 3 seconds target with intelligent caching)
- Clear risk warnings for user safety
- Enterprise-grade reliability with comprehensive error handling

## System Architecture

### UI/UX Decisions
The platform adheres to Material Design-inspired principles with a data-centric approach, utilizing a blue color scheme (#2563EB primary), Inter font for body text, and JetBrains Mono for monospace. Design emphasizes data clarity, functional hierarchy, responsive grid layouts, minimal animations, and accessibility. Key components include a search interface, token overview card, liquidity panel, risk analysis card, token holders table (ranked list with clickable addresses), live recent transfers section, and enterprise AI summary section. A blockchain explorer with multi-page routing for addresses, tokens, and transactions is included.

### Technical Implementations

#### Core Infrastructure
The system features a full-stack implementation with a schema-first approach. The frontend is built with React, TypeScript, Tailwind CSS, Shadcn UI, and TanStack Query. The backend uses Express.js with enterprise-grade multi-chain support via ethers.js (EVM) and @solana/web3.js (Solana). All data is fetched live from blockchain sources.

#### Enterprise Systems
1.  **RPC Failover System:** Multi-endpoint health checks with automatic failover, supporting primary + fallback RPC providers per chain, ensuring graceful degradation and zero downtime.
2.  **Enterprise Logo Resolution:** Guarantees zero broken images using a multi-CDN fallback chain (TrustWallet, CoinGecko API, Uniswap token lists) with SVG avatar generation and Identicon fallback. Includes intelligent caching and automatic format selection.
3.  **Live Market Data Integration:** Uses CoinGecko API (primary) and DexScreener API (fallback) for real-time prices, market cap, volume, and circulating supply detection.
4.  **Historical Price Data System:** Fetches real 24h price history from CoinGecko `/market_chart` endpoint (hourly data points) with DexScreener fallback (current price). Features 5-minute caching, SVG visualization with gradient fills, division-by-zero protection for single-point data, and decoupled rendering from current price availability. Supports both EVM and Solana tokens.
5.  **Live Transaction Scanner:** Real-time EVM event log scanning via blockchain RPC with automatic event decoding, smart contract method detection, and gas price analysis.
6.  **Real-Time Holder Scanner (OPTIMIZED):** Live holder distribution from blockchain RPC with dual-mode performance optimization:
    * **APPROXIMATE MODE** (default, recommended for free RPC tiers): Uses delta calculation from Transfer events only, avoiding balanceOf() batch queries. Fast: 2-5 seconds for 50K blocks. Prevents RPC rate limits on Alchemy free tier.
    * **ACCURATE MODE** (optional, requires generous RPC limits): Delta calculation + balanceOf() verification for all addresses. Slower: 15-30 seconds, may hit rate limits.
    * Features: Adaptive chunking, BigInt precision, holder categorization (Whale, Large, Medium, Small). (EVM only)
7.  **Token Flow Analyzer:** Analyzes token inflow/outflow from live blockchain Transfer events over multiple time periods, calculating net flow, transfer counts, and unique addresses. (EVM only)
8.  **Enhanced Risk Analysis:** Provides 100% blockchain-verified risk assessment including contract age, proxy pattern detection, trading tax detection, honeypot detection, unlimited minting detection, owner privilege scanning, liquidity risk assessment, and holder concentration analysis.
9.  **Enhanced AI Summarizer:** Integrates Google Gemini AI 2.5 Flash for comprehensive on-chain pattern detection, professional investor insights, and anomaly highlighting.
10. **Enhanced Metadata Aggregator:** Multi-source metadata aggregation system that merges data from CoinGecko API with on-chain and market data. Features include:
    * **Comprehensive Token Data**: Categories, websites, social links (Twitter, Telegram), token description (500 char truncated), fully diluted valuation (FDV)
    * **Type-Safe Design**: All nullable fields normalized to `undefined` (never `null`), arrays default to `[]`, ensuring strict schema compliance
    * **Multi-Source Priority**: On-chain data (most reliable) > Market data > CoinGecko metadata
    * **URL Validation**: Deduplication and cleaning for website links, proper social handle extraction
    * **Frontend Display**: Categories as badges, websites with external link icons (max 3 shown), social links with brand icons, formatted price/FDV display
    * **Performance**: CoinGecko API cached for 5 minutes, graceful fallback on API failures
11. **DApp Activity Scanner:** Production-ready blockchain activity analyzer that scans recent blocks for token Transfer events, groups transactions by destination contract to identify active DApps, and calculates total transaction counts and gas spent per contract. Features include:
    * **Adaptive Block-Range Detection:** Automatically detects RPC provider limits (e.g., Alchemy free tier 10-block limit) by parsing error messages and adjusting chunk size dynamically (2000 → 50 → 10 blocks).
    * **While-Loop Cursor System:** Explicit cursor tracking ensures zero coverage gaps when chunk size adapts mid-scan. Cursor advances only on successful fetch, preventing block skipping during retries.
    * **Intelligent Degraded Mode:** Marks scans as degraded only when coverage <100% due to rate limiting or early termination. Full scans show no degraded warnings even if throttled during execution.
    * **Accurate Coverage Math:** Uses inclusive block ranges (latestBlock - fromBlock + 1) with defensive double-check before returning results, guaranteeing coveragePercent never exceeds 100%.
    * **User-Friendly Messaging:** Surfaces partial data scenarios with specific messages ("~X% coverage", "rate limited", "limited data") while maintaining silent success for complete scans.
    * **Enterprise Reliability:** Rate limiting backoff (500ms-4s exponential), 5-consecutive-error tolerance, transaction fetching with 100-item limit, gas analysis for each contract interaction.

#### Multi-Chain Architecture
Features a multi-chain dispatcher with automatic chain detection, dedicated EVM workers, and Solana integration. A robust caching layer (NodeCache) with intelligent TTLs is used for various data types.

### Feature Specifications
-   **Live Data Features (100% Blockchain-Verified):** Search interface, Price & Flow Metrics Panel (live price, 24H change, 24H/12H/4H token inflow/outflow, net flow), **24h Historical Price Chart** (SVG visualization with real CoinGecko/DexScreener data, automatic caching, fallback handling), **Enhanced Token Metadata** (categories as badges, websites with external links, Twitter/Telegram social links, token description, FDV display), live market data, liquidity panel (multi-DEX aggregation), enhanced risk analysis (blockchain-verified score), **Token Holders Table** (ranked list with address, quantity, percentage - live blockchain data), live recent transfers, **DApp Activity** (top smart contracts by transaction count and gas spent), enhanced AI summary.
-   **Blockchain Explorer:** Clickable addresses with dedicated routing for `/address/:addr`, `/tx/:hash`, and `/token/:addr`.
-   **UI Layout:** Compact single-page design with all sections visible. Token Overview + Metric Cards (left column), Token Details (center), 24h Price Chart (right). Full-width sections for Liquidity + Risk Analysis, Token Holders table, DApp Activity table, and Recent Transfers table. Responsive layout with loading skeletons, degraded state badges, and comprehensive error states.

### System Design Choices
-   **Enterprise Reliability:** Multi-layer fallback systems, comprehensive error handling, intelligent caching, RPC endpoint health monitoring, and zero single points of failure.
-   **Data Authenticity:** All data derived from live blockchain sources, ensuring no mock values or synthetic data.
-   **Performance Optimization:** Parallel RPC calls, strategic caching, lazy loading, and efficient BigInt arithmetic.

## External Dependencies

### APIs & Services
-   **CoinGecko API**: Primary market data (price, volume, market cap, circulating supply), historical price data (`/market_chart` endpoint for 24h hourly data points), and enhanced metadata via `/coins/{platform}/contract/{address}` endpoint (categories, websites, social links, description, FDV).
-   **DexScreener API**: Fallback market data, liquidity scanning, and historical price fallback (current price as single data point).
-   **Google Gemini AI**: AI-powered summaries (gemini-2.5-flash model). Requires `GEMINI_KEY` environment variable.
-   **Alchemy API**: Priority 1 RPC provider for Ethereum, Base, Polygon, Optimism, Arbitrum, and Solana. Requires `ALCHEMY_API_KEY` environment variable (✅ SUDAH DIISI).
-   **Etherscan Family APIs**: Contract verification, token metadata, and holder counts for multiple chains:
    * **Etherscan** (Ethereum): Requires `ETHERSCAN_API_KEY` (✅ SUDAH DIISI)
    * **BscScan** (BSC): Requires `BSCSCAN_API_KEY` (⚠️ ISI MANUAL di Replit Secrets)
    * **PolygonScan** (Polygon): Requires `POLYGONSCAN_API_KEY` (⚠️ ISI MANUAL di Replit Secrets)
    * **Arbiscan** (Arbitrum): Requires `ARBISCAN_API_KEY` (⚠️ ISI MANUAL di Replit Secrets)
    * **Optimistic Etherscan** (Optimism): Requires `OPTIMISTIC_ETHERSCAN_API_KEY` (⚠️ ISI MANUAL di Replit Secrets)
    * **BaseScan** (Base): Requires `BASESCAN_API_KEY` (⚠️ ISI MANUAL di Replit Secrets)
    * **SnowTrace** (Avalanche): Requires `SNOWTRACE_API_KEY` (⚠️ ISI MANUAL di Replit Secrets)
-   **Solscan API**: Solana blockchain explorer for verification and metadata. Requires `SOLSCAN_API_KEY` (✅ SUDAH DIISI).
-   **TrustWallet Assets CDN**: Primary token logo source.
-   **CoinGecko Logo API**: Secondary token logo source.
-   **Uniswap Token Lists**: Tertiary token logo source.

### Blockchain Infrastructure
-   **RPC Endpoints**: Configured via Replit Secrets for various EVM chains (Ethereum, Base, BSC) and Solana.
-   **ethers.js**: For EVM chain interactions.
-   **@solana/web3.js**: For Solana chain interactions.

### Infrastructure
-   **NodeCache**: In-memory caching.
-   **Express.js**: API server.
-   **React + TypeScript**: Frontend development.