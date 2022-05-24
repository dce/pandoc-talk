# Pandoc: a Tool I Use and Like

**David Eisinger**  
Dev Team Meeting, 5/24/2022

---

# Pandoc: a Tool I Use and Like

What is [Pandoc][1]? From the project website:

> If you need to convert files from one markup format into another, pandoc is your swiss-army knife.

---

# Pandoc: a Tool I Use and Like

* I spend a lot of time writing,
  * and I love [Vim][3], [Markdown][4], and the command line
  * (and avoid browser-based WYSIWYG editors when I can)
* But it has a ton of utility outside of that
  * Really, anywhere you need to move between different text-based formats, Pandoc can probably help

---

## Markdown ➞ Craft Blog Post

I do all my blog writing in Vim/Markdown, then, when I'm ready to publish, convert to HTML with Pandoc, then paste that into a Craft text block.

This gets me a few things I really like:

* Curly quotes in place of straight ones and en-dashes in place of `--` (from the [`smart` extension][9])
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

I also sometimes find myself writing decently long [Basecamp][11] posts Basecamp 3 has a fine WYSIWYG editor (🪦 Textile), but again, I'd rather be in Vim. Pasting HTML into Basecamp doesn't work (just shows the code verbatim), but I've found that if I convert my Markdown notes to HTML and open the HTML in a browser, I can copy and paste that directly into Basecamp with good results. Leveraging MacOS' `open` command, this one-liner does the trick[^2]:

```
cat [filename.md] \
  | pandoc -t html \
  > /tmp/output.html \
  && open /tmp/output.html \
  && read -n 1 \
  && rm /tmp/output.html
```

This will convert the contents to HTML, save that to a file, open the file in a browser, wait for the user to hit enter, and the remove the file. Without that `read -n 1`, it'll remove the file before the browser has a chance to open it.

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

We built an app for one of our clients that takes in news articles (in HTML) via an API and sends them as emails to _their_ clients (think big brands) if certain criteria are met. Recently, we were making improvements to the plain text version of the emails, and we noticed that some of the articles were coming in without any linebreaks in the content. When we removed the HTML (via Rails' [`strip_tags` helper][12]), the resulting content was all on one line, which wasn't very readable. So imagine an article like this:

```
<h1>Headline</h1> <p>A paragraph.</p> <ul><li>List item #1</li> <li>List item #2</li></ul>
```

Our initial approach (with `strip_tags`) gives us this:

```
Headline A paragraph. List item #1 List item #2
```

Not great! But fortunately, some bright fellow had the idea to pull in Pandoc, and some even brighter person packaged up some [Ruby bindings][2] for it. Taking that same content and running it through `PandocRuby.html(content).to_plain` gives us:

```
Headline

A paragraph.

-   List item #1
-   List item #2
```

Much better, and though you can't tell from this basic example, Pandoc does a great job with spacing and wrapping to generate really nice-looking plain text from HTML.

---

## HTML Element ➞ Text

A few months ago, we were doing Pointless Weekend and needed a domain for our [Thrillr][13] app. A few of us were looking through lists of fun top-level domains, but we realized that AWS Route 53 only supports a limited set of them. In order to get everyone the actual list, I needed a way to get all the content out of an HTML `<select>` element, and you'll never guess what I did (unless you guessed "use Pandoc"). In Firefox:

* Right click the select element, then click "Inspect"
* Find the `<select>` in the DOM view that pops up
* Right click it, then go to "Copy", then "Inner HTML"
* You'll now have all of the `<option>` elements on your clipboard
* In your terminal, run `pbpaste | pandoc -t plain`

The result is something like this:

```
.ac - $76.00
.academy - $12.00
.accountants - $94.00
.agency - $19.00
.apartments - $47.00
.associates - $29.00
.au - $15.00
.auction - $29.00
...
```

---

## Preview Mermaid/Markdown (`--standalone`)

A different client recently asked for an architecture diagram of a complex system that [Andrew][14] and I were working on, and we opted to use [Mermaid][15] (which is rad BTW) to create sequence diagrams to illustrate all of the interactions. Both GitHub and GitLab support Mermaid natively, which is really neat, but we wanted a way to quickly iterate on our diagrams without having to push changes to the remote repo.

We devised a simple build chain ([demo version available here][16]) that watches for changes to a Markdown file, converts the Mermaid blocks to SVG, and then uses Pandoc to take the resulting document and convert it to a styled HTML page using the `--standalone` option ([here's the key line][18]). Then we could simply make our changes and refresh the page to see our progress.

---

## Generate a PDF

* Pandoc also includes several ways to create PDF documents.
* The simplest (IMO) is to install `wkhtmltopdf`
  * then instruct Pandoc to convert its input to HTML but use `.pdf` in the output filename

```bash
echo "# Hello\n\nIs it me you're looking for?" \
  | pandoc -t html -o hello.pdf
```

---

## Closing Thoughts

I think that's about all I have to say about Pandoc for today. A couple final thoughts:

* Pandoc is incredibly powerful -- I've really only scratched the surface here. Look at the [man page][5] to get a sense of everything it can do.
* Pandoc is written in Haskell, and [the source][8] is pretty fun to look through if you're a certain kind of person.

So install Pandoc with your package manager of choice and give it a shot. I think you'll find it unexpectedly useful.

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
