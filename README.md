# Jet's Wiki

Served with GitHub Pages at https://jetpaine.github.io/jets-wiki. GitHub Pages builds it with Jekyll automatically; there's no build step to run.

- `index.html`: the wiki home page.
- `defender-x/`: the Defender X sub-wiki, one Markdown file per page. Footnotes are written `[^name]` in the text and `[^name]: …` at the bottom.
- `_layouts/dx.html`: the page template for the Defender X pages (sidebar, header, footer).
- `_data/dx_nav.yml`: the Defender X sidebar. Add new pages here.
- `assets/dx.css`: the Defender X styles.

Each Defender X page starts with a few settings between `---` lines: `title`, `summary`, `status`, and `checked` (the date the page was last compared with the game's code). Sub-articles also set `parent` and `parent_url`.

To preview locally: `gem install jekyll kramdown-parser-gfm`, then `jekyll serve` and open http://localhost:4000/jets-wiki/.
