---
date: '2026-07-25T12:57:06+02:00'
draft: false
title: 'Oh my glyph'
---

## Intro

Can you tell what will be printed in each of these Golang print statements without running them?

```golang
  fmt.Println("-- Exhaling Face Emoji --")
  fmt.Println("In Quotes: 😮‍💨")
  fmt.Println(`In Backticks: 😮‍💨`)
  fmt.Println("Unicode In Quotes: \U0001f62e\u200d\U0001f4a8")
  fmt.Println("Unicode Double-Escaped: \\U0001f62e\\u200d\\U0001f4a8")
  fmt.Println(`Unicode In Backticks: \U0001f62e\u200d\U0001f4a8`)
```

{{<details summary = "See Answer">}}

Here's what they print on the [Go Playground](https://go.dev/play/p/-S9cq23ad55):

```text
-- Exhaling Face Emoji --
In Quotes: 😮‍💨
In Backticks: 😮‍💨
Unicode In Quotes: 😮‍💨
Unicode Double-Escaped: \U0001f62e\u200d\U0001f4a8
Unicode In Backticks: \U0001f62e\u200d\U0001f4a8
```

{{< /details >}}
<br/>

If you were able to tell, you can stop reading now! If not, let's dig deeper into the plumbing a *glyph* goes through before getting rendered on a screen.

## The Components

The final thing you visually see on the screen is called a *glyph*. The final rendered glyph can differ from one device to another. For example, my Windows PC draws some emojis differently than my MacBook because each platform ships different fonts. Even in source code, you can write a literal string `\U0001f62e\u200d\U0001f4a8` in your text editor, and still see an exhaling face 😮‍💨 when issuing a print statement on that string literal. It can be kind of deceiving, but there are actually multiple layers that affect the final glyph due to conversions throughout the stack and differences between platforms.

## Basics

### ASCII and Unicode

In the beginning of time, there was ASCII. It was simple, and it worked well with typewriters. ASCII defined 128 characters: English letters, digits, punctuation, and some control characters like newline (`\n`) and tab (`\t`). Since ASCII was a 7-bit encoding, every character fit into a single byte, making text processing very straightforward.

Applications of that time mostly dealt with bytes. The text renderer at the end was responsible for mapping those bytes into visible characters. This renderer could be a physical typewriter, or a terminal emulator drawing pixels on a screen. Control characters like `\t` weren't drawn at all. Instead, they instructed the renderer to move the cursor.

Unsurprisingly, not all text can be represented by 128 characters. We need to represent other languages `白`, mathematical symbols `ø`, emojis `😮‍💨`, and much more. Thus, Unicode was created. Unicode is a much larger character set that assigns a unique **code point** to every character. Unicode itself does not define how these code points are stored as bytes. This is left to encoding standards.

### UTF-*

UTF-8 and UTF-16 are example encodings of Unicode. UTF-8 is what I'll describe here, and it's what Golang source files and strings use, so it matches the example above.

UTF-8 encodes Unicode code points into one or more bytes. For example, the letter `A` is encoded as the single byte `0x41`, while the Chinese character `白` is encoded as the three-byte sequence `0xE7 0x99 0xBD`.

The leading byte indicates how long the UTF-8 sequence is, while continuation bytes always begin with the bit pattern `10`. This allows applications to determine where each encoded code point begins and ends.

## The Golang example

### Literal emoji

If we look back at our example print statements one by one:

```golang
fmt.Println("Literal Exhaling In Quotes: 😮‍💨")
```

This prints the exhaling face emoji 😮‍💨.

What can be potentially tricky is: thinking only about the Golang compiler/runtime without considering an important component: the text editor we're writing the literal in! The flow is actually as follows:

1. The text editor is able to render 😮‍💨 visually. In the editor's memory or on disk, it is actually stored as the UTF-8 byte sequence:

   ```
   0xF0 0x9F 0x98 0xAE
   0xE2 0x80 0x8D
   0xF0 0x9F 0x92 0xA8
   ```

   Those bytes encode the following Unicode code points:

   - U+1F62E (Face with Open Mouth)
   - U+200D (Zero Width Joiner)
   - U+1F4A8 (Dashing Away)

2. When the Golang compiler reads the source file, it simply reads those bytes from the source code and embeds the same byte sequence into the compiled binary. It does not recognize that the byte sequence represents Unicode characters nor it does anything special.

3. At runtime, `fmt.Println` writes those bytes to `os.Stdout`.

4. Those bytes keep getting passed across application layers until they eventually reach the terminal or text editor displaying the output. It decodes the UTF-8 byte sequence back into the three Unicode code points above.

5. A text shaping engine and font determine how those code points should be displayed. If the font supports this emoji sequence, the three code points are rendered as the single glyph 😮‍💨 (Side note: some text renderers might [not support](#zwj) this and instead display it as two glyphs 😮💨.)

This is the exact same flow when using backticks instead of double quotes:

```golang
fmt.Println(`Literal Exhaling In Backticks: 😮‍💨`)
```

Raw string literals disable escape processing. But there are no escape sequences here anyway, it's all just bytes! The compiler simply embeds the bytes exactly as they appear in the source file.

I.e., in both examples, the Golang text editor does the heavy lifting of Unicode parsing, storing, rendering, and UTF-8 encoding in steps (1) and (5).

### Literal Unicode escapes

The third print statement was:

```golang
fmt.Println("Exhaling Unicode: \U0001f62e\u200d\U0001f4a8")
```

Visually we see a sequence of literal ASCII characters, but the output is still the exhaling emoji 😮‍💨! This time, the Golang compiler is the one interpreting the Unicode escapes rather than the text editor.

1. The text editor stores the literal characters:

   ```
   \ U 0 0 0 1 f 6 2 e \ u 2 0 0 d \ U 0 0 0 1 f 4 a 8
   ```

   Since these are ordinary ASCII characters, this sequence occupies 26 bytes in the UTF-8 source file.

2. When the Golang compiler encounters a double-quoted string, it interprets `\U` and `\u` sequences. It recognizes the following Unicode points during compilation:

   - U+1F62E
   - U+200D
   - U+1F4A8

3. The compiler then encodes those code points into UTF-8:

   ```
   U+1F62E → 0xF0 0x9F 0x98 0xAE
   U+200D  → 0xE2 0x80 0x8D
   U+1F4A8 → 0xF0 0x9F 0x92 0xA8
   ```

   and embeds the resulting byte sequence into the compiled binary:

   ```
   0xF0 0x9F 0x98 0xAE
   0xE2 0x80 0x8D
   0xF0 0x9F 0x92 0xA8
   ```

The rest of the steps are then exactly the same as before:

4. At runtime, `fmt.Println` writes those bytes to `os.Stdout`.

5. The output application decodes the bytes back into the three Unicode code points.

6. The text shaping engine and font combine them into the single glyph 😮‍💨.

The escape sequences themselves never make it into the compiled binary, they are resolved by the compiler into Unicode code points, which are then encoded as bytes.  The main difference is that this time the Golang compiler was responsible for the byte-encoding, rather than the text editor.

This is **not** the case for:

```golang
fmt.Println("Literal Unicode Exhaling In Quotes: \\U0001f62e\\u200d\\U0001f4a8")
fmt.Println(`Literal Unicode Exhaling In Backticks: \U0001f62e\u200d\U0001f4a8`)
```

In the first case, the compiler sees `\\` and knows the backslash itself has been escaped, so it does **not** begin a Unicode escape sequence.

In the second case, raw string literals perform no escape processing whatsoever.

In both cases, the literal ASCII characters

```
\U0001f62e\u200d\U0001f4a8
```

are embedded into the compiled binary exactly as written (26 ASCII bytes), written to stdout, and eventually rendered as literal text rather than an emoji.

Hopefully it's now more intuitive to connect the printed glyphs to the underlying encoding at different parts of the stack!

## Bonus Tips

### One glyph, multiple Unicode code points {#zwj}

One thing you may have noticed is that the exhaling emoji actually consisted of multiple Unicode codepoints!

A useful mental model is to think of the pipeline like this:

```
UTF-8 bytes
      ↓
Unicode code points
      ↓
Grapheme cluster(s)
      ↓
Glyph(s)
```

The three code points

- U+1F62E
- U+200D
- U+1F4A8

form a single **grapheme cluster**, which most users perceive as a single "character."

The Zero Width Joiner (ZWJ) tells the text shaping engine that these emoji should be treated as a single emoji sequence rather than independent emoji.

If the renderer or font doesn't support this sequence, you'll instead see two separate emoji, like:

```
😮💨
```

rather than the single exhaling face 😮‍💨.

### Different encoding, same visuals

Unicode introduces another challenge: the same visible text can sometimes be represented by different underlying code points.

For example, `é` can be represented as either:

- the single code point `U+00E9`, or
- the two code points `U+0065` (`e`) followed by `U+0301` (combining acute accent).

Although they produce the same glyph, their UTF-8 byte sequences are different.

This is why comparing two strings byte-for-byte can sometimes produce surprising results, leading to bugs like "logging in from my iPhone works, but logging in from my Windows PC doesn't." In situations like these, Unicode normalization is often needed before comparison.

### Screen readers

Sometimes when thinking about text, people only consider visuals, but not everyone consumes content visually!

A [non-negligible number](https://webaim.org/projects/screenreadersurvey10/) of internet users rely on screen readers to consume content.

So if you decide to use fancy-looking glyphs like:

```
𝔭𝔯𝔢𝔱𝔱𝔶 𝔱𝔢𝔵𝔱
```

a screen reader won't necessarily read that as "pretty text."

Instead, it will typically identify those Unicode characters individually, saying things like:

> "Mathematical fraktur small p... mathematical fraktur small r... mathematical fraktur small e..."

Depending on the screen reader, language, and accessibility settings, the exact wording may differ. Either way, decorative Unicode characters can significantly reduce accessibility, and it's one thing to keep in mind when building accessible products.