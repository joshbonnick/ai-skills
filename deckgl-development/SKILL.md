---
name: deckgl-development
description: Assist with creating, optimizing and explaining deck.gl visualizations. Use when the user requests deck.gl layer code, examples, performance tips, or integration patterns.
---

# DeckGL Development

## When to Apply

- Always invoke when the user's message relates to map, map rendering or GeoJSON.
- Generating new deck.gl layers (Scatterplot, Column, Arc, GeoJson, etc.) for specific datasets or map regions.
- Explaining integration patterns with React, Mapbox, or MapLibre.
- Providing performance guidance for large datasets, e.g., batching, GPU acceleration, or efficient layer updates.
- Troubleshooting rendering or interaction issues, such as tooltips, picking, or z-index conflicts.
- Suggesting best practices for visual clarity, color schemes, and data encoding.
- Translating user requirements or data formats into fully working deck.gl code snippets.

## Guidelines

When generating code:

- Output valid JavaScript/TypeScript
- Comment on performance considerations
- Include basic data loading examples
- Include usage of `initialViewState` and `layers`

### 1. Performance → `rules/performance.md`

- Performance should be a top priority.
- Use `deck.log.enable()` to log performance metrics.
- Use `deck.log.profile()` to profile performance.
- Use `deck.log.time()` to time code blocks.
- Use `deck.log.assert()` to check for performance issues.
