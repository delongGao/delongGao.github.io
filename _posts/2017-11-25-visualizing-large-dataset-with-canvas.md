---
layout: default
title:  "Visualizing Large Data set using Canvas"
date:   2017-11-25 16:16:01 -0600
categories: working_journals canvas
---

# Journal #1 Visualizing Large Data set using Canvas

## The Problem
### Background

Sequences alignment result could be treated as a `m * n` 2-d array of bases("letters"), where `m` is the number of sequences being aligned and `n` is the number of bases of the aligned sequence. `n` is also going to be the length of the longest sequence of the original ones, because the shorter ones from the original sequences will be supplemented with `"-"`.

### Potential Difficulties

- We need to have a good experience / performance for potentially large `m` and `n`
- Single base needs to have individual visualization and interaction

Consider the following alignment:
```
m = 1000
n = 1600
m * n = 1,600,000
```
Rendering this many elements all together and keeping each of them interactive on the frontend could be very performance intensive, it's also going to be a very heavy load for the initial request.

## Approaches
### Render all bases at once

- Pros:
  - After initial loading and rendering, everything is cached on the frontend, so theoretically following interaction should have almost no cost.

- Cons:
  - Initial loading time is very long, possible crashing
  - Heavy work on frontend
    - D3: too many elements rendered on the page
    - Canvas: the generated static image could be too large to load (over 10mb for around 300 sequences)

### Render partial(active) view with an additional buffer

The following is a partial loading example: 
- Moving active view within the boundaries of buffer view will not trigger reloading
- Once active view reaches the boundary of the current buffer view, reload happens and a new active view with its buffer views will be generated

![partial loading example](/assets/images/visualizing-large-dataset-with-canvas/visual_partial_loading.png "Partial Loading Example")

- Pros:
  - Less initial loading and rendering time
  - Could scale up with `m` and `n` with consistent performance

- Cons:
  - Need to make additional requests whenever user moves to a new active area
  - We discard previous calculation results when the new active area is rendered

Based on my observations, the benefits of additional requests and partial rendering overweigh the overhead; plus it scales better with increasing alignment size.

To elaborate on the idea further, here are some logic diagram and some code samples for the essential methods:
![code logic](/assets/images/visualizing-large-dataset-with-canvas/code_logic.png "Code Logic")

- `scroll_handler`

```javascript
scroll_handler: function() {
  // other scroll operations...
  var current_position = AlignmentSequences.current_position();
  AlignmentVisualPositions.update_active_boundary(current_position);
  
  if (AlignmentVisualPositions.reach_buffer_boundary()) {
    var current_position = AlignmentSequences.current_position();
    var scroll_direction = AlignmentSequences.scroll_direction();
    if (scroll_direction === 'horizontal') {
      AlignmentVisualPositions.update_buffer_boundary(current_position);
      SequenceLogo.refresh();
    } else {
      AlignmentVisualPositions.update_buffer_vertical_boundary(current_position);
    }
    AlignmentSequences.refresh();
  }
}
```

- `reach_buffer_boundary`

```javascript
reach_left_boundary: function() {
  var active_boundary = AlignmentVisualPositions.current_active_boundary,
      buffer_boundary = AlignmentVisualPositions.current_buffer_boundary;

  return active_boundary.horizontal_start <= buffer_boundary.horizontal_start &&
            active_boundary.horizontal_start > AlignmentVisualPositions.alignment_length_min;
}
// similar functions for each of the four directions...

// calling the functions defined above
reach_buffer_boundary: function() {
  return AlignmentVisualPositions.reach_left_boundary() ||
            AlignmentVisualPositions.reach_right_boundary() ||
            AlignmentVisualPositions.reach_top_boundary() ||
            AlignmentVisualPositions.reach_bottom_boundary();
}
```

- `update_active_boundary`

```javascript
update_active_horizontal_boundary: function(position) {
  var active_boundary = AlignmentVisualPositions.current_active_boundary;

  active_boundary.horizontal_start = position.horizontal_start;
  active_boundary.horizontal_mid = position.horizontal_mid;
  active_boundary.horizontal_end = position.horizontal_end;
},
// similar function for updating vertical boundary...

update_active_boundary: function(position) {
  AlignmentVisualPositions.update_active_horizontal_boundary(position);
  AlignmentVisualPositions.update_active_vertical_boundary(position);
}
```

- `update_buffer_boundary`

```javascript
update_buffer_horizontal_boundary: function(position) {
  AlignmentVisualPositions.update_active_horizontal_boundary(position);

  var active_boundary = AlignmentVisualPositions.current_active_boundary,
      h_start = active_boundary.horizontal_start,
      h_end = active_boundary.horizontal_end,
      h_window_delta = h_end - h_start + 1,
      buffer_boundary = AlignmentVisualPositions.current_buffer_boundary;

  // we also need to make sure the buffer boundary doesn't go beyond the valid range
  buffer_boundary.horizontal_start = Math.max(AlignmentVisualPositions.alignment_length_min, h_start - h_window_delta);
  buffer_boundary.horizontal_end = Math.min(AlignmentVisualPositions.alignment_length_max, h_end + h_window_delta);
}

update_buffer_boundary: function(position) {
  // max number of positions for a single sequence
  AlignmentVisualPositions.alignment_length_max = EntropyHeatmap.bucket_count;
  AlignmentVisualPositions.sequence_count_max = AlignmentSequences.sequence_count;

  AlignmentVisualPositions.update_buffer_horizontal_boundary(position);
  AlignmentVisualPositions.update_buffer_vertical_boundary(position);
},
```

- `refresh`

```javascript
refresh: function() {
  // other operations to setting up loading environment...
  // loading_alignment_visual is just sending a ajax call 
  //    to get the json data from back end
  load_alignment_visual({
    active_boundary: AlignmentVisualPositions.current_active_boundary,
    buffer_boundary: AlignmentVisualPositions.current_buffer_boundary,
    visual_type: 'alignment_sequences'
  });
},
```


## Future Work
### Additional Request Timing
Trigger loading earlier before reaching the buffer boundary so that user doesn't have to wait for the loading animation to get the next active view
### Lighter Request Payload
Instead of re-rendering all the new active view every time, only loading and rendering the additional content. This could make the additional request payload much smaller, but will require to change the rendering logic to be able to handle both full view loading and partial view loading.
### Code refactor
All the functionalities right now require a lot of javascript code, the current implementation is not well structured and organized in a object-oriented manner. For example, all the different visual class and shared visual position related functionalities could benefit a lot if we could use inheritance and interfaces. 

One possible direction: Typescript. :)

## Lessons Learnt
Canvas could be a viable option: more static, less elements on the page

Implementing canvas is very different and has its pitfalls:
  - it's static in nature, but could be made dynamic with some techniques
  - ??? the performance penalties of not rendering properly. NOTE: I'm not sure how to expand this properly.
  - not enough documentations, the ones I have found very useful are [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D) and some of the html5rocks posts.

It's very important to consider the trade-offs between caching large data on frontend vs. loading smaller data from back end more frequently.