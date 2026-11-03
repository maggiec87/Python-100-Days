name: Publish notebooks to GitHub Pages

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Convert notebooks to HTML
        run: |
          python -m pip install nbconvert
          python - <<'PY'
          from pathlib import Path
          from urllib.parse import quote
          import html
          import subprocess

          root = Path(".")
          output = Path("site")
          notebooks = sorted(
              path for path in root.rglob("*.ipynb")
              if ".git" not in path.parts and "site" not in path.parts
          )

          links = []
          for notebook in notebooks:
              relative = notebook.relative_to(root)
              destination = output / relative.parent
              destination.mkdir(parents=True, exist_ok=True)

              subprocess.run(
                  [
                      "jupyter", "nbconvert", "--to", "html",
                      "--output-dir", str(destination),
                      str(notebook),
                  ],
                  check=True,
              )

              html_path = relative.with_suffix(".html").as_posix()
              links.append(
                  f'<li><a href="{quote(html_path, safe="/")}">'
                  f'{html.escape(relative.as_posix())}</a></li>'
              )

          output.mkdir(parents=True, exist_ok=True)
          (output / "index.html").write_text(
              "<!doctype html><meta charset='utf-8'>"
              "<title>Python-100-Days</title>"
              "<h1>Python-100-Days</h1><ul>"
              + "\n".join(links)
              + "</ul>",
              encoding="utf-8",
          )
          PY

      - uses: actions/configure-pages@v5

      - uses: actions/upload-pages-artifact@v3
        with:
          path: site

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
