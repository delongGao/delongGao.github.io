---
layout: default
title:  "Visualizing Large Data set using Canvas"
date:   2017-11-25 16:16:01 -0600
categories: working_journals canvas
---

### Journal #1 Visualizing Large Data set using Canvas

- What's the problem
  - why bother?
  - why hard? example?
  - assumptions? constraints?

- Approaches
  - 1: d3, loading whole
  - 2: canvas, loading whole
  - 3: our approach based on previous experiments
    - highlights
    - one full working scenario example
    - how we solve the issues in previous approaches

- future work
  - reduce refresh cycle
  - smoother refreshing mechanism and interactions
  - code refactor

- lessons learnt
  - canvas could be a viable option: more static, less elements on the page
  - implementing canvas is very different and has its pitfalls:
    - it's static in nature, but could be made dynamic with some techniques
    - ??? the performance penalties of not rendering properly. NOTE: I'm not sure how to expand this properly
    - not enough documentations, the ones I have found very useful are [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D) and some of the html5rocks posts
  - it's very important to consider the trade-offs between caching large data on frontend vs. loading smaller data from backend more frequently