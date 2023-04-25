---
theme: uncover
hidden: true
style: |
  .center_img {
    margin: auto;
    display: block;
  }

  .side_by_side {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
  }

  .small-text {
    font-size: 0.75rem;
    letter-spacing: 1px;
    font-family: "Times New Roman", Tahoma, Verdana, sans-serif;
  }
  li {
    font-size: 28px;
    letter-spacing: 1px;
  }
  p.quote {
    line-height: 38px;
  }
  q {
    font-size: 32px;
    letter-spacing: 1px;
  }
  cite {
    text-align: right;
    font-size: 28px;
    margin-top: 12px;
    margin-bottom: 128px;
  }
paginate: true
backgroundColor: #FFFFFF
marp: true
---

## LiveView

<img class="center_img" src="assets/concurrency_vs_parallelism.png" width="40%" />

---

### Съдържание

1. Основи на уеб програмирането.
2. HTTP протокол.
3. Plug

---

### LiveView Loop

LiveView will receive events, like link clicks, key presses, or page submits.
• Based on those events, you’ll write functions to transform your state.
• After you change your state, LiveView will re-render only the portions of
the page that are affected by the transformed state.
• After rendering, LiveView again waits for events, and we go back to the
top

---

### LiveViews == Behavior

* Behaviour is the opposite of a library
* When using a library, your code is in charge and calls the library
* When using a behaviour, the behaviour is in charge and calls parts of your code

---

### The React Idea

* Collocate the highly coupled: template, state and the functions that and the funcitons
  that operate on them