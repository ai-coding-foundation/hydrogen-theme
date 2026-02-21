---
id: hatch3r-performance-budgets
type: rule
description: Performance budgets and targets for the project
scope: always
---

# Performance Budgets

| Metric                          | Budget             |
| ------------------------------- | ------------------ |
| UI render                       | 60fps (16ms/frame) |
| Cold start to interactive       | 1.5 seconds        |
| Idle CPU usage                  | 1%                 |
| Memory footprint                | 30 MB              |
| Event processing latency        | 10ms per event     |
| Bundle size (initial, gzipped)  | 500 KB             |
| Backend reads per session start | ≤ 5 documents      |
| Serverless warm execution       | 500ms              |
