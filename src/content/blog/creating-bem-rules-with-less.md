---
title: "Generating BEM Rules with LESS"
description: "Edit (2017-05-19): This was a horrible idea and you should absolutely not do this. Much better to keep the code flat so it is easier to search"
pubDate: "2015-05-15T00:00:00"
---

**Edit (2017-05-19): This was a horrible idea and you should absolutely not do this. Much better to keep the code flat so it is easier to search**

BEM is a great way to build styles that work well with components. For work, we are using LESS for preprocessing. While working on building some reusable components, I stumbled upon a nice pattern for utilizing nesting to build BEM style rules. Let's look at a quick example how we can write our styles in LESS using nesting and generate a nice flat BEM style result.

LESS:

```
.block {
    font-size: 14px;

    &__element-one {
        color: black;

        &--modifier-one {
            color: blue;
        }

        &--modifier-two {
            color: red;
        }
    }

    &__element-two {
        color: grey;

        &--modifier-one {
            color: green;
        }

        &--modifier-two {
            color: yellow;
        }
    }
}
```

Generated CSS:

```
.block {
  font-size: 14px;
}
.block__element-one {
  color: black;
}
.block__element-one--modifier-one {
  color: blue;
}
.block__element-one--modifier-two {
  color: red;
}
.block__element-two {
  color: grey;
}
.block__element-two--modifier-one {
  color: green;
}
.block__element-two--modifier-two {
  color: yellow;
}
```

This simple pattern allows us to avoid duplicating the block and elements for each of the nested elements and modifiers respectively and the resulting CSS is nice and clean.
