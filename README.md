# howellyang.com

Source for [www.howellyang.com](https://www.howellyang.com): an online CV and blog built with [Hugo](https://gohugo.io).

## Updating the CV

Almost everything on the homepage comes from one file: **`data/cv.yaml`**.

| Section on the page | Key in `data/cv.yaml` |
| --- | --- |
| Name, photo, title, links, bio, interests | `profile` |
| News | `news` |
| Experience | `experience` (each job has `groups` of bullet `items`) |
| Selected Projects | `projects` (use `video`, `image`, or `metric` + `metric_label` for the left-hand visual) |
| Patent, talks | `patents_talks` |
| Skills | `skills` |
| Education | `education` |

Text fields accept Markdown (`**bold**`, `[link](https://...)`). To add a PDF CV, put it in `static/files/` and set `profile.cv_pdf`, for example `/files/YangHao_CV.pdf`.

## Chinese edition (/zh/)

The site is bilingual. The Chinese CV at `/zh/` uses the same template as the English homepage, with its content in **`data/cv_zh.yaml`** (same structure as `data/cv.yaml`; keep the two in sync). Chinese project pages live in `content/zh/projects/`, one per English page with the **same file name**, so the 中文 / EN switch in the header lands on the matching page.

- Interface labels for both languages: `data/strings.yaml`
- A page is Chinese when its front matter has `ui_lang: zh` (Chinese project pages also need `type: projects`)
- The old `/about/` page now redirects to `/zh/`; `content/fixed/about.md` is kept as a draft and can be deleted
- In `cv_zh.yaml`, write each Chinese paragraph on one line: YAML line folding inserts spaces between Chinese characters

Blog posts are in `content/post/` and are not translated.

## Layout and style

- `layouts/index.html` (English) and `layouts/cv/list.html` (Chinese): both render `layouts/partials/cv_body.html`
- `layouts/projects/`: project detail pages and the projects index
- `layouts/shortcodes/`: `pipeline` (flow diagrams) and `callout` (highlighted notes) for project pages
- `layouts/partials/head.html`, `site_header.html`, `default_head.html`, `default_foot.html`: shared page frame
- `layouts/post/single.html`: blog post page
- `static/css/site.css`: all styles, including a mobile layout and a print layout (printing the homepage gives a clean paper CV)

The site uses system fonts only, so it loads quickly everywhere, including mainland China. The old Lanyon theme in `themes/` is still referenced for a few fallback list templates, but its styles are no longer loaded.

## Build

```sh
hugo server          # preview at http://localhost:1313
hugo                 # build into ./output
```

## License

The source files in this repository are licensed under the MIT License. The generated pages and the content (text, images, videos) are not.
