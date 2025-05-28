# Future Skill wiki
This is the public repository for Future Skills' wiki.
The main purpose of the wiki is to act as a development resource when making new exercises/challenges/tournaments/freecodes for [futureskill.com](https://futureskill.com).

The wiki is based on [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/). Please refer to that page for documentation of the wiki framework.
In a nutshell, each wiki page is a Markdown document with some additional functionality. Check [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) for more in-depth information.

All Python dependencies can be found in `requirements.txt`.
You can install them easily with `pip3 install -r requirements.txt`, and then run a preview server for the wiki with the command `mkdocs serve`.

Additionally, you will need to install Cairos Graphics, see instructions [here](https://squidfunk.github.io/mkdocs-material/plugins/requirements/image-processing/).
Alternatively, you can (temporarily) disable the `social` plugin in `mkdocs.yml` while working locally, just remember to re-enable it afterwards.

## Adding a new document
You add a new document by adding a markdown file to one of the folders under `docs` other than `docs/assets`.
You can add more folders or sub-folders if needed, just remember to update the `.gitignore` with `!docs/<new folder>` and `!docs/<new folder>/*.md` if you do.
You will also need to add your new document to the `nav` structure in `mkdocs.yml`.
Depending on where you place it, it will show up under different tabs, and could even have a new subsection under that tab.

At the top of the new document, you should have a section that looks like this:
```
---
title: <title for the document>
<other metadata>
---
```
This will set the name of the page, and also the top-level title in the document corresponding to `# <title for the document>`.
To ensure the rest of the titles format correctly, it is recommended that you do not use any single hash titles, but instead start with: `## <next title>`.

If you want to reference another wiki page from your new markdown file, you can easily do so like this: `[<link text>](<relative path>/<document name>.md)`.
You can further optionally add the title of a section to the link, like this: `[<link text>](<relative path>/<document name>.md#<section title>)`.

**Example:**
Let's say that we want to add a reference to the `Overview` section in the `Data` page that is in the `create_an_exercise` folder, from a document in the `basics` folder, then we would write like this:

"... see the [Overview sections on the Data page](../create_an_exercise/Data.md#overview) for more information."

## Adding new images
To add a new image, you simply add it to `/docs/assets`.
If you add a file in a format that has not been added before, you also need to update the `.gitignore` with the line `!docs/assets/*.<file format>`.

Once you have the image stored in `assets`, you can reference it from a document like this: `![](../assets/<file name>.<file format>)`, assuming that your document is in one of the `docs` child folders.

## Configuring the wiki
You edit the `mkdocs.yml`-file to configure the wiki.
Everything before a colon is a key, with the value coming after the colon.
If something is indented, it is a child of whatever is the closest thing above it that is indented one step to the left.

## Change the URL
If you want to change the URL, you will need to set the `site_url` value in the `mkdocs.yml` file to the new URL, replace the URL in the `CNAME` file with the new URL, and update the DNS service.
