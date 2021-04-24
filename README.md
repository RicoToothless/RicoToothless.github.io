# Hugo Usage

## quick start

add new article
```bash
hugo new post/{categories}/{article}.md
```

> WARNING: If the date is in the future, it may miss articles

serve local website
```bash
hugo serve -D
```

## add new images

add new images in directory `static`

and insert image in article

```markdown
![gitbook-login.png](/gitbook-login.png)
```

## deploy to github repo & github page

My Github page type is User/Organization Pages. And git commit would trigger GitHub Action.
