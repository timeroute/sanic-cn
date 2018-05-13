# 贡献

谢谢你的关注！Sanic 一直在寻找贡献这。如果你不习惯贡献代码，那么向源文件中添加文档是非常值得赞赏的。

## 安装

要开发sanic（主要是为了运行测试），强烈建议从源代码安装。

所以假设你已经克隆了repo，并且已经在已经设置了虚拟环境的工作目录中，然后运行：

```bash
python setup.py develop && pip install -r requirements-dev.txt
```

## 运行测试

要运行 sanic 的测试，强烈建议使用 tox 如下：

```bash
tox
```

看就这么简单！

## Pull requests!

pull request 批准规则非常简单：
1. 所有 pull requests 必须通过测试
* 所有 pull requests 必须至少被项目中的一名当前合作者
* 所有 pull requests 必须通过 flake8 检查
* 如果你决定从任何公共接口中 删除/修改 任何内容，则应附上该消息。
* 如果你执行一个新的功能你应该至少有意个单元测试来附上它。

## 文档

Sanic 的文档使用 [sphinx](http://www.sphinx-doc.org/en/1.5.1/) 建立。指南用 Markdown 协程，并且能在 `docs` 目录中找到，这个模块参考由 `sphinx-apidoc` 自动生成。

要从头生成文档：

```bash
sphinx-apidoc -fo docs/_api/ sanic
sphinx-build -b html docs docs/_build
```

HTML 文档会在 `docs/_build` 目录创建。

## 警告

Sanic 的主要目标之一是速度。代码可能会降低 Sanic 的性能，但在可用性，安全性或功能方面没有显着提升，可能不会合并。请不要被这吓到！如果您对某个想法有任何疑虑，开个 issue 来讨论和帮助吧。
