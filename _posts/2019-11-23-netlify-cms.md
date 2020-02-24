---
layout: post
---

https://github.com/netlify/netlify-cms

Enable users to sign in with Github and create posts online
https://www.netlifycms.org/docs/add-to-your-site/#authentication

# Configur

```
# admin/config.yml
backend:
  name: git-gateway
  branch: master

publish_mode: editorial_workflow

media_folder: "_images"
public_folder: "assets/images"

collections:
  - name: "blog"
    label: "Blog"
    folder: "_posts"
    create: true 
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
    # identifier_field: latin
    editor:
      preview: false
    fields:
      - {label: "Layout", name: "layout", widget: "hidden", default: "post"}
      - {label: "Title", name: "title", widget: "string"}
      - {label: "Latin", name: "latin", widget: "string", hint: 'Ovaj se koristi za URL posto ћирилица nije uobicajena za linkove. Kad se postavi, vise se ne moze promenuti'}
      - {label: "Publish Date", name: "date", widget: "date"}
      - {label: "Author", name: "author", widget: "select", options: ['vladan', 'slobodan', 'dragutin', 'darko', 'sreten', 'branislav']}
      - {label: "Body", name: "body", widget: "markdown"}
```

# Domain

You can use netlify dns so it can automatically add domains.
https://docs.netlify.com/domains-https/custom-domains/configure-external-dns/
For external DNS you need to add CNAME that point to netlify.com subdomain like
`practical-pare-68a238.netlify.com`
```
```
