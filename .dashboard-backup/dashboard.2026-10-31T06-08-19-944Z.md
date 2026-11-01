---
dashboard: true
layout: immersive
banner:
  quote: "The mind is everything. What you think you become."
  author: "Buddha"
  image: "https://images.unsplash.com/photo-1788651946293-893d6b217f34?q=80&w=1374&auto=format&fit=crop&ixlib=rb-4.1.0"
  quoteFont: "Didot,\"Bodoni MT\",Georgia,serif"
  images:
    - "https://images.unsplash.com/photo-1788651946293-893d6b217f34?q=80&w=1374&auto=format&fit=crop&ixlib=rb-4.1.0"
immersiveBackground: "https://images.unsplash.com/photo-1788651946293-893d6b217f34?q=80&w=1374&auto=format&fit=crop&ixlib=rb-4.1.0"
immersive:
  - id: "widget:quickActions"
    w: 3
    cap: 18
    fixed: true
    x: 9
    y: 10
  - id: "widget:weather"
    w: 3
    cap: 19
    fixed: true
    x: 0
    y: 17
  - id: "widget:calendar"
    w: 3
    cap: 33
    fixed: true
    x: 0
    y: 36
  - id: "widget:habit"
    w: 2
    cap: 19
    fixed: true
    x: 10
    y: 58
  - id: "widget:pomodoro"
    w: 2
    cap: 14
    fixed: true
    x: 10
    y: 28
  - id: "widget:expense"
    w: 2
    cap: 16
    fixed: true
    x: 10
    y: 42
  - id: "widget:countdown-cd-2026-end"
    w: 3
    cap: 10
    fixed: true
    x: 9
    y: 0
  - id: "widget:music"
    w: 3
    cap: 17
    fixed: true
    x: 0
    y: 0
  - id: "card:demo-memo-1"
    w: 3
    cap: 21
    fixed: true
    x: 3
    y: 0
  - id: "card:demo-memo-path"
    w: 4
    cap: 21
    fixed: true
    x: 6
    y: 32
  - id: "card:demo-todo-1"
    w: 3
    cap: 32
    fixed: true
    x: 6
    y: 0
  - id: "card:demo-todo-2"
    w: 3
    cap: 32
    fixed: true
    x: 3
    y: 21
  - id: "section:Library"
    w: 7
    cap: 24
    fixed: true
    x: 3
    y: 53
  - id: "widget:skills"
    w: 3
    cap: 12
    fixed: true
columns:
  - name: Sticky Notes
    color: "#f59e0b"
    type: sticky
  - name: Library
    color: "#8b5cf6"
    type: projects
    showCover: false
    library:
      viewMode: grid
      sortBy: "modified"
      sortDesc: true
---

## Sticky Notes

### 2026-10-31 memo
id: demo-memo-1
type: generic
Welcome to Apex Dashboard! Click here to edit your first memo.

### Tips
id: demo-memo-path
type: generic
- Capture a thought in the top bar and press Enter to save it
- Drag tiles anywhere — they snap to nearby edges
- Right-click a tile to remove it or tidy the layout; drag its corner to resize
- The + button (top-left) adds memos, todos, sections and widgets

### Getting Started
id: demo-todo-1
type: task
- [ ] Capture a quick thought in the top bar
- [ ] Drag this card somewhere else
- [ ] Grab the card's corner and make it bigger
- [ ] Open the top-left + and add a widget

### Interface Guide
id: demo-todo-2
type: task
- [ ] Switch or create workspaces from the top-left pill
- [ ] Enable weather, expense, music and more in Settings
- [x] Right-click any tile: remove, tidy the layout
- [x] Click the edit button by the clock to change the poster

## Library

### Reading
id: demo-lib-reading
type: project

### To Read
id: demo-lib-toread
type: project

### Done
id: demo-lib-done
type: project
