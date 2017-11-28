---
layout: default
title:  "Visualizing Large Data set using Canvas"
date:   2017-11-25 16:16:01 -0600
categories: working_journals canvas
---

### Journal #1 Visualizing Large Data set using Canvas

- The Problem
  - Background

    Sequences alignment result could be treated as a `m * n` 2-d array of bases("letters"), where `m` is the number of sequences being aligned and `n` is the number of bases of the aligned sequence. `n` is also going to be the length of the longest sequence of the original ones, because the shorter ones from the original sequences will be supplemented with `"-"`.

  - Potential Difficulties?

    - We need to have a good experience / performance for potentially large `m` and `n`
    - Single base needs to have individual visualization and interaction
    
    For a large 1000 sequences alignment with the longest sequence of 1600 bases, the total bases will be 1,600,000. Rendering this many elements all together and keeping each of them interactive on the frontend could be very performance intensive, it's also going to be a very heavy load for the initial request.

- Approaches
  - Render all bases at once

    Pros:
    - After initial loading and rendering, everything is cached on the frontend, so theoretically following interaction should be lighter and easier.

    Cons:
    - Initial loading time is very long, possibility of crashing the page
    - Heavy work on frontend
      - D3: too many elements rendered on the page
      - Canvas: the generated static image could be too large to load (over 10mb for around 300 sequences)

  - Only Render partial of the bases that user are actively viewing

    The following is a partial loading example: 
    - Moving active view within the boundaries of buffer view will not trigger reloading
    - Once active view reaches the boundary of one of buffer view, reload happens and a new active view with its buffer views will be generated

    ![partial loading example](/assets/images/visual_partial_loading.png "Partial Loading Example")

    Pros:
    - Less initial loading and rendering time
    - Could scale up with `m` and `n` with consistent performance

    Cons:
    - Need to make additional requests whenever user moves to a new active area
    - We discard previous calculation results when the new active area is rendered

    Based on my observations, the benefits of additional requests and partial rendering overweigh the overhead; plus it scales better with increasing alignment size.

- future work
  - reduce refresh overhead
  - smoother refreshing mechanism and interactions
  - code refactor

- lessons learnt
  - canvas could be a viable option: more static, less elements on the page
  - implementing canvas is very different and has its pitfalls:
    - it's static in nature, but could be made dynamic with some techniques
    - ??? the performance penalties of not rendering properly. NOTE: I'm not sure how to expand this properly
    - not enough documentations, the ones I have found very useful are [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D) and some of the html5rocks posts
  - it's very important to consider the trade-offs between caching large data on frontend vs. loading smaller data from backend more frequently