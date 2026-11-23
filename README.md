# System Design Interview Notes — HLD, LLD & Case Studies

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

System design interview notes: HLD and component Q&A (databases, Redis, Kafka, REST), distributed-systems basics, real case studies, full designs and LLD / machine-coding problems. The `01_insider_guide_Alex_Xu` folder has chapter-by-chapter notes on the book *System Design Interview* by Alex Xu (Vol 1 and Vol 2). The book itself is not part of this repo.

**59 notes**, each ranked by how often the topic comes up in backend interviews. Written in plain English for readers whose first language is not English.

## 🚀 Start here

The most-asked topics (🔥🔥). Read these first:

1. [🎯 LLD & Machine Coding — The Playbook](04_lld_and_machine_coding/00_lld_interview_playbook.md)
2. [Ch 1: Scale From Zero to Millions of Users](01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md)
3. [Distributed Systems — The Fundamentals](02_distributed_systems/01_distributed_fundamentals.md)
4. [🅿️ Machine Coding / LLD — Parking Lot System](04_lld_and_machine_coding/01_parking_lot.md)
5. [⚡ Design a Flash Sale System](05_designs/02_flash_sale_system.md)
6. [🛗 Machine Coding LLD — Elevator System (Lift)](04_lld_and_machine_coding/03_elevator_system.md)
7. [🏟️ Design Ticketmaster (Reserved-Seat Event Ticketing)](05_designs/03_ticketmaster_event_ticketing.md)
8. [Ch 4: Design a Rate Limiter](01_insider_guide_Alex_Xu/vol_1/ch04_design_a_rate_limiter.md)
9. [Ch 4: Distributed Message Queue](01_insider_guide_Alex_Xu/vol_2/ch04_distributed_message_queue.md)
10. [🛵 Design Swiggy / Zomato (Food Delivery: Orders, Dispatch, Live Tracking)](05_designs/04_food_delivery_swiggy_zomato.md)

## 🧭 How to use this repo

Every note has a **priority**. It says how often the topic comes up in interviews, and so how early you should study it.

| Priority | Meaning | Notes |
|---|---|:---:|
| 🔥🔥 Very high | Asked in almost every interview. Study first. | 25 |
| 🔥 High | Asked often. Study next. | 19 |
| ⚡ Medium | Asked sometimes. Good to know. | 11 |
| 💤 Low | Rarely asked. Read only if you have time. | 3 |
| — | Not rated yet. | 1 |

Inside a note, the same icons appear next to each section heading, so you can also pick the important sections of a long note.

## 📚 All notes

### 📁 iq

| Priority | Note |
|---|---|
| 🔥🔥 Very high | [Kafka Interview Questions](iq/04_Kafka_Interview_Questions.md) |
| 🔥🔥 Very high | [System Design HLD Interview Questions](iq/01_System_Design_HLD_Interview_Questions.md) |
| 🔥🔥 Very high | [Database Design and Internals Interview Questions](iq/02_Database_Design_and_Internals_Interview_Questions.md) |
| 🔥 High | [Redis Interview Questions](iq/03_Redis_Interview_Questions.md) |
| 🔥 High | [REST API Design Interview Questions](iq/05_REST_API_Design_Interview_Questions.md) |
| ⚡ Medium | [Elasticsearch — Step by Step](iq/06_Elasticsearch_Interview_Questions.md) |

### 📁 00_interview_toolkit

| Priority | Note |
|---|---|
| — | [🧰 Problem → Fix — The Pre-Round Cheat Sheet](00_interview_toolkit/01_problem_to_fix_cheatsheet.md) |

### 📁 01_insider_guide_Alex_Xu/vol_1

| Priority | Note |
|---|---|
| 🔥🔥 Very high | [Ch 1: Scale From Zero to Millions of Users](01_insider_guide_Alex_Xu/vol_1/ch01_scale_from_zero.md) |
| 🔥🔥 Very high | [Ch 4: Design a Rate Limiter](01_insider_guide_Alex_Xu/vol_1/ch04_design_a_rate_limiter.md) |
| 🔥🔥 Very high | [Ch 5: Design Consistent Hashing](01_insider_guide_Alex_Xu/vol_1/ch05_design_consistent_hashing.md) |
| 🔥🔥 Very high | [Ch 6: Design a Key-Value Store](01_insider_guide_Alex_Xu/vol_1/ch06_design_a_key_value_store.md) |
| 🔥🔥 Very high | [Ch 8: Design a URL Shortener](01_insider_guide_Alex_Xu/vol_1/ch08_design_url_shortener.md) |
| 🔥🔥 Very high | [Ch 10: Design a Notification System](01_insider_guide_Alex_Xu/vol_1/ch10_design_notification_system.md) |
| 🔥🔥 Very high | [Ch 11: Design a News Feed System](01_insider_guide_Alex_Xu/vol_1/ch11_design_news_feed_system.md) |
| 🔥🔥 Very high | [Ch 12: Design a Chat System](01_insider_guide_Alex_Xu/vol_1/ch12_design_a_chat_system.md) |
| 🔥 High | [Ch 2: Back-of-the-Envelope Estimation](01_insider_guide_Alex_Xu/vol_1/ch02_back_of_envelope_estimation.md) |
| 🔥 High | [Ch 7: Design a Unique ID Generator in Distributed Systems](01_insider_guide_Alex_Xu/vol_1/ch07_design_unique_id_generator.md) |
| 🔥 High | [Ch 13: Design a Search Autocomplete System](01_insider_guide_Alex_Xu/vol_1/ch13_design_search_autocomplete_system.md) |
| 🔥 High | [Ch 14: Design YouTube](01_insider_guide_Alex_Xu/vol_1/ch14_design_youtube.md) |
| ⚡ Medium | [Ch 3: A Framework for System Design Interviews](01_insider_guide_Alex_Xu/vol_1/ch03_framework_for_system_design_interviews.md) |
| ⚡ Medium | [Ch 9: Design a Web Crawler](01_insider_guide_Alex_Xu/vol_1/ch09_design_web_crawler.md) |
| ⚡ Medium | [Ch 15: Design Google Drive](01_insider_guide_Alex_Xu/vol_1/ch15_design_google_drive.md) |

### 📁 01_insider_guide_Alex_Xu/vol_2

| Priority | Note |
|---|---|
| 🔥🔥 Very high | [Ch 4: Distributed Message Queue](01_insider_guide_Alex_Xu/vol_2/ch04_distributed_message_queue.md) |
| 🔥🔥 Very high | [Ch 7: Hotel Reservation System](01_insider_guide_Alex_Xu/vol_2/ch07_hotel_reservation_system.md) |
| 🔥🔥 Very high | [Ch 11: Payment System](01_insider_guide_Alex_Xu/vol_2/ch11_payment_system.md) |
| 🔥🔥 Very high | [Ch 12: Digital Wallet](01_insider_guide_Alex_Xu/vol_2/ch12_digital_wallet.md) |
| 🔥 High | [Ch 1: Proximity Service](01_insider_guide_Alex_Xu/vol_2/ch01_proximity_service.md) |
| 🔥 High | [Ch 2: Nearby Friends](01_insider_guide_Alex_Xu/vol_2/ch02_nearby_friends.md) |
| 🔥 High | [Ch 5: Metrics Monitoring and Alerting System](01_insider_guide_Alex_Xu/vol_2/ch05_metrics_monitoring_alerting.md) |
| 🔥 High | [Ch 6: Ad Click Event Aggregation](01_insider_guide_Alex_Xu/vol_2/ch06_ad_click_event_aggregation.md) |
| 🔥 High | [Ch 10: Real-time Gaming Leaderboard](01_insider_guide_Alex_Xu/vol_2/ch10_real_time_gaming_leaderboard.md) |
| 🔥 High | [Ch 13: Stock Exchange](01_insider_guide_Alex_Xu/vol_2/ch13_stock_exchange.md) |
| ⚡ Medium | [Ch 3: Google Maps](01_insider_guide_Alex_Xu/vol_2/ch03_google_maps.md) |
| ⚡ Medium | [Ch 9: S3-Like Object Storage](01_insider_guide_Alex_Xu/vol_2/ch09_s3_like_object_storage.md) |
| 💤 Low | [Ch 8: Distributed Email Service](01_insider_guide_Alex_Xu/vol_2/ch08_distributed_email_service.md) |

### 📁 02_distributed_systems

| Priority | Note |
|---|---|
| 🔥🔥 Very high | [Distributed Systems — The Fundamentals](02_distributed_systems/01_distributed_fundamentals.md) |
| 🔥 High | [🏗️ Load Balancing Strategies — Justifying, Not Naming](02_distributed_systems/02_load_balancing_strategies.md) |

### 📁 03_case_studies

| Priority | Note |
|---|---|
| 🔥 High | [🏛️ Stack Overflow — The Monolith That Scaled](03_case_studies/01_stack_overflow_architecture.md) |
| ⚡ Medium | [🎮 Discord — Cassandra → ScyllaDB](03_case_studies/02_discord_cassandra_to_scylladb.md) |
| ⚡ Medium | [📺 Prime Video — Serverless Back to a Monolith](03_case_studies/03_prime_video_monolith_vs_serverless.md) |

### 📁 04_lld_and_machine_coding

| Priority | Note |
|---|---|
| 🔥🔥 Very high | [🎯 LLD & Machine Coding — The Playbook](04_lld_and_machine_coding/00_lld_interview_playbook.md) |
| 🔥🔥 Very high | [🅿️ Machine Coding / LLD — Parking Lot System](04_lld_and_machine_coding/01_parking_lot.md) |
| 🔥🔥 Very high | [🛗 Machine Coding LLD — Elevator System (Lift)](04_lld_and_machine_coding/03_elevator_system.md) |
| 🔥🔥 Very high | [🚦 Low Level Design of a Rate Limiter — Sliding Window, Leaky Bucket & Factory Pattern](04_lld_and_machine_coding/05_rate_limiter.md) |
| 🔥🔥 Very high | [🎟️ Machine Coding / LLD — BookMyShow Seat Booking (with Seat Locking)](04_lld_and_machine_coding/08_bookmyshow_seat_booking.md) |
| 🔥🔥 Very high | [💸 Machine Coding / LLD — Splitwise (Expense Sharing)](04_lld_and_machine_coding/09_splitwise.md) |
| 🔥 High | [🥤 Low Level Design of a Vending Machine — The Canonical State Pattern Problem](04_lld_and_machine_coding/02_vending_machine.md) |
| 🔥 High | [🔐 Low Level Design of an Inventory Locker System — Amazon / Warehouse Management](04_lld_and_machine_coding/04_inventory_locker_system.md) |
| 🔥 High | [🏗️ Low Level Design of a Cache — From Scratch](04_lld_and_machine_coding/07_lru_cache.md) |
| ⚡ Medium | [📋 Low Level Design of a Process Logger — Google Interview Question (Part 1)](04_lld_and_machine_coding/06_process_logger.md) |
| ⚡ Medium | [🎲 Machine Coding / LLD — Snake & Ladder](04_lld_and_machine_coding/10_snake_and_ladder.md) |
| ⚡ Medium | [✈️ Low Level Design of MakeMyTrip — Flight Booking System](04_lld_and_machine_coding/51_flight_booking_modelling.md) |
| 💤 Low | [♟️ Low Level Design of a Chess App — From Scratch](04_lld_and_machine_coding/50_chess_modelling.md) |
| 💤 Low | [⭕ Low Level Design of Tic Tac Toe Game](04_lld_and_machine_coding/99_tic_tac_toe.md) |

### 📁 05_designs

| Priority | Note |
|---|---|
| 🔥🔥 Very high | [⚡ Design a Flash Sale System](05_designs/02_flash_sale_system.md) |
| 🔥🔥 Very high | [🏟️ Design Ticketmaster (Reserved-Seat Event Ticketing)](05_designs/03_ticketmaster_event_ticketing.md) |
| 🔥🔥 Very high | [🛵 Design Swiggy / Zomato (Food Delivery: Orders, Dispatch, Live Tracking)](05_designs/04_food_delivery_swiggy_zomato.md) |
| 🔥 High | [🛍️ Design a Product Catalog Platform](05_designs/01_product_catalog_platform.md) |
| 🔥 High | [🏢 Design a Multi-Tenant SaaS API Platform](05_designs/05_multi_tenant_saas_api_platform.md) |

## 🔗 More Similar Notes

- **[DSA](https://github.com/vikas9dev/dsa-algo)** — data structures & algorithms, LeetCode patterns
- **[Java & Spring Boot](https://github.com/vikas9dev/java)** — Java and Spring interview questions, design patterns
- **[AWS](https://github.com/vikas9dev/aws/tree/system_design)** — AWS services for system design interviews
- **[claude-job-hunt](https://github.com/vikas9dev/claude-job-hunt)** — Claude Code skills for a software-engineering job search: find roles, tailor a resume, debrief interviews

## ⚠️ Disclaimer

These are personal study notes. They are checked with care, but they can still have mistakes. Check important facts in the official documentation.

The `01_insider_guide_Alex_Xu` folder holds **unofficial** study notes on *System Design Interview — An Insider's Guide* (Vol 1 and Vol 2) by Alex Xu (and Sahn Lam for Vol 2). They are not affiliated with or endorsed by the authors or ByteByteGo, and they do not replace the book. If the notes help you, please buy the book.

## 🤝 Contributing

Found a mistake? Open an issue with the **Correction** form, or send a pull request. See [CONTRIBUTING.md](CONTRIBUTING.md) and the [Code of Conduct](CODE_OF_CONDUCT.md).

## 📄 License

[CC BY-NC-SA 4.0](LICENSE). You may share and adapt these notes **with credit**, **not for commercial use**, and under the **same license**.

If this repo helps you, please give it a ⭐ so that other people can find it too.
