# Hugo Usage

## add new content

```bash
hugo new post/{categories}/{article}.md
```

## add new images

add new images in directory `static`

and insert image in article

```markdown
![gitbook-login.png](/gitbook-login.png)
```

## deploy to github repo & github page

my github page type is User/Organization Pages

```bash
./deploy.sh '{your git commit message}'
```
