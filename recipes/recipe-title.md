---
title: Recipe Title
description: Recipe Description
hidden: false
recipe:
  color: '#018FF4'
  icon: 🦉
---
```shell Shell
// Start a new repo in a new folder (you can also use an existing repo)
mkdir [your repo name]
git init
// Add your spec! Example: https://petstore.swagger.io/v2/swagger.json
git add -A

// GitHub Action in .github/workflows/readme.yml
name: Sync OAS to ReadMe
on:
  push:
    branches:
      - master
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: readmeio/github-readme-sync@v2
        with:
          readme-oas-key: ${{ secrets.README_OAS_KEY }}
           
          # OPTIONAL CONFIG, use if necessary
          # oas-file-path: './[your spec name].json'
          # api-version: 'v1.0.0'

```

```json Response Example
{"success":true}
```

# test

<!-- shell@ -->

