---
layout: post
title: "Github Actions报告"
date: 2023-05-28 
description: "Github Actions报告"
tag: 报告
---   

# 01任务描述 

实现将 Markdown 文件自动转换成 PDF 文件的功能。
集成该功能至 GitHub Actions 中，使其自动在每次 push 时执行该转换任务。

# 02 解决方案

安装 pandoc 和 LaTeX

pandoc 是一个开源的文档转换工具，可以将 Markdown 文件转换成多种格式，包括 HTML、PDF、Word、ePub 等。而 LaTeX 则是一种文本排版系统，常用于生成高质量的科技论文和学术报告。因此，我们需要先安装 pandoc 和 LaTeX。
```    
$ sudo apt-get update
$ sudo apt-get install pandoc texlive-full
```   

编写转换脚本
为了实现将 Markdown 文件转换成 PDF 文件的功能，我们需要编写一段转换脚本。这里我们使用 pandoc 来执行转换，并使用 xelatex 编译生成的 PDF 文件，因为它支持中文。

``` 
#!/bin/bash
# Convert markdown to pdf using pandoc and xelatex

if [ -z "$1" ]
  then
    echo "No argument supplied"
    exit 1
fi

INPUT_FILE=$1
OUTPUT_FILE="${INPUT_FILE%.*}.pdf"

pandoc $INPUT_FILE \
--pdf-engine xelatex \
--template eisvogel \
--output $OUTPUT_FILE
```
# 03 代码说明

首先，我们检查是否传入了输入文件名。
接着，将输入文件名和输出文件名保存到变量中。
最后，使用 pandoc 进行转换，使用 xelatex 编译生成的 PDF 文件，并使用 eisvogel LaTeX 模板进行排版。
集成至 GitHub Actions
接下来，我们需要将脚本集成到 GitHub Actions 中，在 push 时自动执行该任务。我们可以将上述转换脚本编写为一个 GitHub Actions Workflows，如下所示：
```
name: have a try

on:
  workflow_dispatch:
  push:
    branches:
      - 'main'
    paths:
      - '.github/workflows/**.yml'
      - 'doc/**'
    tags:        
      - v*             # Push events to v1 tag

jobs:
  convert:
    if: ${{ !startsWith(github.ref, 'refs/tags/') }}
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2 # access repo files
      - name: PandocLatex
        uses: ./
        with:
          args: >-
            pandoc doc/test1.md 
            -o output/test1.pdf
            --from markdown 
            --resource-path=assets 
            --number-sections 
            --top-level-division=chapter 
            --shift-heading-level-by=-1 
            --highlight-style monochrome 
            --pdf-engine "xelatex" 
            --toc 
            -V CJKmainfont="Noto Sans Mono CJK SC"
            -V toc-own-page="true" 
            -V book
            -V code-block-font-size="\normalsize"
            
      - name: UploadArtifact
        uses: actions/upload-artifact@master
        with:
          name: build-${{ github.run_number }}
          path: artifact

      - name: PushCommit
        if: ${{ !startsWith(github.ref, 'refs/tags/') }}
        run: |
          git config user.name nicole01101101zke
          git config user.email 19966422680@163.com
          git add .
          git commit -m "CI: produce build${{ github.run_number }}.pdf"
          git push

  release:
    if: startsWith(github.ref, 'refs/tags/')
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
      - name: Release
        uses: softprops/action-gh-release@v1
        with:
          files: artifact/build.pdf
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
首先，我们定义了该 workflow 的名称和触发事件为 push，仅在 main 分支上触发。
然后，在作业中使用 Ubuntu 系统，并执行以下步骤：
检出代码
安装 pandoc 和 LaTeX
执行转换脚本
上传转换生成的 Markdown 和 PDF 文件作为 artifacts。

# 04 总结
通过这个 pipeline，clone repository 后 push，自动触发 GitHub Actions Workflow，在 Ubuntu 环境中安装 pandoc 和 LaTeX，执行转换脚本，然后上传转换后的 markdown 文件和 PDF 文件作为 artifacts。这个实现可以让用户更加方便地将 Markdown 文件转换成 PDF 文件，并且在 push 新代码时，能够自动更新转换生成的 PDF 文件。
