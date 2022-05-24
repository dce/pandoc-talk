# Pandoc: a Tool I Use and Like

**David Eisinger**  
Dev Team Meeting, 5/24/2022

---

# Pandoc: a Tool I Use and Like

What is [Pandoc][1]? From the project website:

> If you need to convert files from one markup format into another, pandoc is your swiss-army knife.

---

# Pandoc: a Tool I Use and Like

* I spend a lot of time writing
  * and I love [Vim][3], [Markdown][4], and the command line
  * (and avoid browser-based WYSIWYG editors when I can)
* But it has a ton of utility outside of that
  * Really, anywhere you need to move between different text-based formats, Pandoc can probably help

---

## Markdown ➞ Craft Blog Post

I do all my blog writing in Vim/Markdown, then, when I'm ready to publish, convert to HTML with Pandoc, then paste that into a Craft text block.

This gets me a few things I really like:

* Curly quotes in place of straight ones and en-dashes in place of `--` (from the [smart extension][9])
* [Daring Fireball-style][10] footnotes with return links

Pandoc uses [Pandoc Markdown][7] by default, an "extended and slightly revised version" of the original syntax.

---

## Markdown ➞ Craft Blog Post

Example post:

```markdown
# Article Title

Here's an article -- it has "smart punctuation" and footnotes[^1].

[^1]: So fancy.
```

Convert to HTML:

```bash
cat examples/blog_post.md | pandoc -t html
```

---

## Markdown ➞ Rich Text (Basecamp)

* I also sometimes find myself writing long [Basecamp][11] posts
* Basecamp 3 has a fine WYSIWYG editor (🪦 Textile)
  * but again, I'd rather be in Vim
* Pasting HTML into Basecamp doesn't work
  * but you can convert MD to HTML
  * open in browser
  * then copy/paste

---

## Markdown ➞ Rich Text (Basecamp)

Example post:

```markdown
* **Team Member #1** is doing great.
* **Team Member #2** is also doing great.
* **Team Member #3** is similarly great.

***

The dev team is great.
```

Generate HTML preview:

```bash
cat examples/team_update.md \
  | pandoc -t html \
  > /tmp/output.html \
  && open /tmp/output.html \
  && read -n 1 \
  && rm /tmp/output.html
```

---

## HTML ➞ Text

* A client app receives news articles (in HTML)
  * and emails them out (as HTML and plain text)
* The incoming articles often lack linebreaks
* Running the article through Rails' `strip_tags` similarly has no linebreaks
* This is unreadable
* Pandoc can convert from HTML to plain text
* And has nice Ruby bindings

---

## HTML ➞ Text

Example article:

```bash
cat examples/article.html
```

Result of `strip_tags`:

```bash
cat examples/article.html \
  | ruby -e "require 'action_controller';
  puts ActionController::Base.helpers.strip_tags(STDIN.read)"
```

With Pandoc:

```bash
cat examples/article.html | pandoc -f html -t plain
```

---

## HTML ➞ Text

In the app:

```ruby
def formatted_plaintext
  @formatted_plaintext ||= PandocRuby.html(full_text).to_plain
end
```

---

## HTML Element ➞ Text

* Working on [Thrillr][13]
* Needed a list of all TLDs available in AWS Route 53
* Options were available in a `<select>` in the AWS console
* You'll never guess what I did
  * (unless you guessed "use Pandoc")

---

## HTML Element ➞ Text

* Right click the select element, then click "Inspect"
* Find the `<select>` in the DOM view that pops up
* Right click it, then go to "Copy", then "Inner HTML"
* You'll now have all of the `<option>` elements on your clipboard

```bash
pbpaste | pandoc -t plain --wrap none | sed 's/00/00\n/g'
```

(This worked without all the `sed` stuff originally, but the dropdown got fancy in the interim.)

---

## Preview Mermaid/Markdown (`--standalone`)

* Andrew and I were creating sequence diagrams with [Mermaid][15]
* GitHub and GitLab both support Mermaid natively
  * But we wanted to be able to quickly iterate on the diagrams
* We devised a simple build chain
  * Watch for changes to a Markdown file
  * Convert the Mermaid blocks to SVG
  * Use Pandoc to take the resulting document and convert it to a styled HTML page using the `--standalone` option

---

## Generate a PDF

* Pandoc also includes several ways to create PDF documents.
* The simplest (IMO) is to install `wkhtmltopdf`
  * then instruct Pandoc to convert its input to HTML
  * but use `.pdf` in the output filename

```bash
echo "# Hello\n\nIs it me you're looking for?" \
  | pandoc -t html -o hello.pdf
```

---

## Closing Thoughts

* Pandoc is incredibly powerful
  * I've really only scratched the surface here
  * Look at the [man page][5] for a sense of everything it can do
* Pandoc is written in Haskell
  * [The source][8] is pretty fun to look through if you're a certain kind of person.

---

## Closing Thoughts

So install Pandoc with your package manager of choice and give it a shot. I think you'll find it unexpectedly useful.

Now I will take questions.

[1]: https://pandoc.org/
[2]: https://github.com/xwmx/pandoc-ruby
[3]: https://www.vim.org/
[4]: https://daringfireball.net/projects/markdown/
[5]: https://manpages.org/pandoc
[6]: https://craftcms.com/
[7]: https://garrettgman.github.io/rmarkdown/authoring_pandoc_markdown.html
[8]: https://github.com/jgm/pandoc/blob/master/src/Text/Pandoc/Readers/Markdown.hs
[9]: https://pandoc.org/MANUAL.html#extension-smart
[10]: https://daringfireball.net/2005/07/footnotes
[11]: https://basecamp.com/
[12]: https://apidock.com/rails/ActionView/Helpers/SanitizeHelper/strip_tags
[13]: https://www.viget.com/articles/plan-a-killer-party-with-thrillr/
[14]: https://www.viget.com/about/team/athomas/
[15]: https://mermaid-js.github.io/mermaid/#/
[16]: https://github.com/dce/mermaid-js-demo
[17]: #
[18]: https://github.com/dce/mermaid-js-demo/blob/main/bin/build#L7=
