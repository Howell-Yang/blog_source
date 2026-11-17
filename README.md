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

The Chinese profile lives in `content/fixed/about.md` (served at `/about/`). Blog posts are in `content/post/`.

## Layout and style

- `layouts/index.html`: the CV homepage
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
