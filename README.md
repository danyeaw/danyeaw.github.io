# Nikola Blog

## Branches

- The src branch is what you edit
- The master branch is split from the source branch to just
regenerate with the output directory from the src branch

## External Content

- Place external content in the files directory on the src branch

## New Post
```bash
$ nikola new_post
```

## Build
```bash
$ nikola build
```
The `-a` option causes a rebuild of everything (even if up to date)

## Check

Check links with:
```bash
$ nikola check -l
```

Check files are generated from src properly with:
```bash
$ nikola check -f
```

# To Generate Slides

Slides are using reveal-md:

```bash
npx reveal-md slides.md --static .
```

