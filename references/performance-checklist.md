# Performance Checklist

Quick-reference checklist with concrete thresholds for performance reviews.
Used by `/review` (Phase 2E), `/eng`, and `/audit`.

## Core Web Vitals Targets

| Metric | Good | Needs Work | Poor |
|--------|------|------------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| INP (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## Budget Thresholds

| Resource | Target |
|----------|--------|
| JavaScript (initial, gzipped) | < 200 KB |
| CSS (gzipped) | < 50 KB |
| API response time (p95) | < 200ms |
| Lighthouse Performance score | ≥ 90 |
| CI pipeline total time | < 10 min |

## Frontend Checks

- [ ] Code splitting and lazy loading for non-critical paths
- [ ] Images optimized (appropriate format, dimensions, lazy-loaded below fold)
- [ ] Explicit dimensions on images and media (prevents CLS)
- [ ] No render-blocking resources in the critical path
- [ ] Main thread JavaScript minimized (long tasks < 50ms)
- [ ] Bundle size monitored in CI (bundlesize, Lighthouse CI)

## Backend Checks

- [ ] No N+1 query patterns (use joins/includes, not loops)
- [ ] List endpoints paginated with enforced limits
- [ ] Database queries use appropriate indexes
- [ ] Caching strategy for stable data (HTTP cache headers, application-level)
- [ ] No unbounded loops or recursive operations on user data
- [ ] Connection pooling for database and external services

## Measurement Discipline

- Measure before optimizing — profiling first, changes second
- Synthetic testing (Lighthouse, DevTools) for reproducible baselines
- Real User Monitoring (web-vitals, CrUX) for production validation
- Performance budgets enforced in CI, not just documented
