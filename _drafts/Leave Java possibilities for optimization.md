How eggs help understand why declarative code is better

Let's compare the next two Java code snippets (consider Python):

```java
for (final Node node : nodes) {
    process(node);
}
```

and

```java
nodes.forEach(this::process);
```

### What is the key difference?

The key difference is that the former is imperative while the latter is declarative.

Аналогия с магазином: "принеси кефиру" vs "сходи в магазин за углом за кефиром". А если кефир уже есть в холодильнике, просто мама об этом забыла? А мама – программист, ты не можешь ей возразить, придётся идти в магазин.

### Which one is better?

Usually such kind of questions are "holy wars" and matter of taste. The second one is one-liner. People like one-liners. But the real reason why it is better is much more important than the number of lines.

Which one would you prefer?
