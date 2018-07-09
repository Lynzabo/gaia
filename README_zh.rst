.. raw:: html
    
    <img src="https://cdn.rawgit.com/michelvocks/ef3894f63c3bb004bca1a2fd5f7eb644/raw/c36d614db8afe229b466b38de1636a82ad809f64/gaia-logo-text.png" width="650px">

|build-status| |go-report| |go-doc| |apache2| |chat|

gaia是一个开源的自动化平台，它使的对于任何编程语言构建强大的pipelines变得既简单又有趣。
。基于`HashiCorp的go-plugin`_ 和 `gRPC`_，gaia是很高效、快速、轻量级和对开发人员友好。Gaia当前还处于alpha版本， `请记住现在先不要使用它构建关键任务。`_

可以使用提供SDKs开发pipeline(目前只支持Go语言SDK)，将代码嵌入git仓库。Gaia会自动克隆您的代码库，将您的代码编译成二进制并按需执行。所有的结果都被流回并格式化为用户友好的图形输出。


动机
==========

.. begin-motivation

*自动化工程师*、*DevOps*、*SRE*、*云工程师*、*平台工程师*——他们都有一个共同点:大多数的技术人员都没有动力去从事这项工作，而且他们很难招聘。

其中一个主要原因是许多自动化工具的抽象和糟糕的执行。它们提供了自己的配置(YAML语法)规范，或者限制用户使用一种特定的编程语言。测试几乎是不可能的，因为大多数自动化工具缺乏模拟服务和子系统的能力。即使是很小的事情，例如解析JSON文件，有时也是非常痛苦的，因为使用了外部过时的库，没有包含在标准框架中，所以自动化解析很麻烦。

We believe it's time to remove all these abstractions and come back to our roots. Are you tired of writing endless lines of YAML-code? Are you sick of spending days forced to write in a language that does not suit you and is not fun at all? Do you enjoy programming in a language you like? Then gaia is for you.
我们认为，是时候抛开所有这些抽象概念，回归本源了。你厌倦了没完没了地写yaml代码吗?你是否厌倦了每天被迫用一种不适合你、一点也不有趣的语言写作吗?你喜欢用自己喜欢的语言编程吗?那么gaia就很适合你。


它是如何工作的呢?
=================

.. begin-architecture

Gaia基于`HashiCorp的go-plugin`，这是一个使用`gRPC`通过HTTP2进行通信的插件系统。HashiCorp最初为开发Packer工具，但是现在也被`Terraform`, `Nomad`和`Vault`大量使用。

Pipelines可以使用任何编程语言编写(gRPC支持是先决条件)，可以在本地编译，也可以简单地在构建系统上编译。Gaia克隆git存储库并自动构建包含的pipeline。

启动一个pipeline后，包含的作业的所有日志输出将返回到gaia并显示在其最终结果状态的详细概述中。

Gaia使用boltDB作为存储，这使得安装步骤非常简单。目前不需要外部数据库。

截图
===========

.. begin-screenshots

|sh-login|
|sh-overview|
|sh-create-pipeline|
|sh-create-pipeline-history|
|sh-pipeline-detailed|
|sh-pipeline-logs|
|sh-settings|

准备开始
===============

.. begin-getting-started

安装
------------

The installation of gaia is simple and often takes a few minutes.
gaia的安装很简单，通常只需要几分钟。

使用docker
~~~~~~~~~~~~

使用以下命令将gaia启动成一个守护进程，将所有数据挂载到当前文件夹。之后，gaia暴露8080端口。使用标准的用户管理和密码管理作为初始登录。建议以后修改密码。

.. code:: sh

    docker run -d -p 8080:8080 -v $PWD:/data gaiapipeline/gaia:latest

手动安装
~~~~~~~~

可以直接在主机系统上安装gaia，在`发布页面`_下载二进制文件。

gaia will automatically detect the folder of the binary and will place all data next to it. You can change the data directory with the startup parameter *--homepath* if you want.
gaia将自动检测二进制文件的文件夹，并将所有数据放在同级目录。您可以使用*--homepath*参数更改默认数据目录位置。

用法
-----

Go
~~~
编写一个pipeline很容易，导入一个库，定义一个函数，该函数将通过一个命令来被gRPC-Server访问。

这是一个例子：

.. code:: go

    package main

    import (
        "log"

	sdk "github.com/gaia-pipeline/gosdk"
    )

    // This is one job. Add more if you want.
    func DoSomethingAwesome() error {
        log.Println("This output will be streamed back to gaia and will be displayed in the pipeline logs.")

	// An error occured? Return it back so gaia knows that this job failed.
	return nil
    }

    func main() {
        jobs := sdk.Jobs{
            sdk.Job{
                Handler:     DoSomethingAwesome,
	        Title:       "DoSomethingAwesome", 
		Description: "This job does something awesome.",

                // Increase the priority if this job should be executed later than other jobs.
		Priority: 0,
	    },
	}

	// Serve
	if err := sdk.Serve(jobs); err != nil {
	    panic(err)
	}
    }

Like you can see, pipelines are defined by jobs. Usually, a function represents a job. You can define as many jobs in your pipeline as you want.

At the end, we define a jobs array that populates all jobs to gaia. We also add some information like a title, a description and the priority. 

The priority is really important and should always be used. If, for example, job A has a higher priority (decimal number) as job B, job A will be executed **after** job B. Priority defines therefore the order of execution. If two or more jobs have the same priority, those will be executed simultanously. You can compare it with the `Unix nice level`_.

That's it! Put this code into a git repository and create a new pipeline via the gaia UI.
Gaia will compile it and add it to it's store for later execution.

Please find a bit more sophisticated example in our `go-example repo`_. 

Roadmap
=======

Gaia is currently in alpha version available. We extremely recommend to not use gaia for mission critical jobs and for production usage. Things will change in the future and essential features may break.

One of the main issues currently is the lack of unit- and integration tests. This is on our to-do list and we are working on this topic with high priority.

It is planned that other programming languages should be supported in the next few month. It is up to the community which languages will be supported next. 

Contributing
============

Gaia can only evolve and become a great product with the help of contributors. If you like to contribute, please have a look at our `issues section`_. We do our best to mark issues for new contributors with the label *good first issue*. 

If you think you found a good first issue, please consider this list as a short guide:

* If the issue is clear and you have no questions, please leave a short comment that you start working on this. The issue will be usually blocked for two weeks to solve it.
* If something is not clear or you are unsure what to do, please leave a comment so we can add further discription.
* Make sure that your development environment is configured and setup. You need `Go installed`_ on your machine and also `nodeJS`_ for the frontend. Clone this repository and run the **make** command inside the cloned folder. This will start the backend. To start the frontend you have to open a new terminal window and go into the frontend folder. There you run **npm install** and then **npm run dev**. This should automatically open a new browser window.
* Before you start your work, you should fork this repository and push changes to your fork. Afterwards, send a merge request back to upstream.

Contact
=======

If you have any questions feel free to contact us on `gitter`_.

.. _`HashiCorp's go-plugin`: https://github.com/hashicorp/go-plugin
.. _`gRPC`: https://grpc.io/
.. _`Do not use it for mission critical jobs yet!`: https://tenor.com/view/enter-at-your-own-risk-gif-8912210
.. _`YAML`: https://en.wikipedia.org/wiki/YAML
.. _`releases page`: https://github.com/gaia-pipeline/gaia/releases
.. _`Packer`: https://www.packer.io/
.. _`Terraform`: https://www.terraform.io/
.. _`Nomad`: https://www.nomadproject.io/
.. _`Vault`: https://www.vaultproject.io/
.. _`boltDB`: https://github.com/coreos/bbolt
.. _`Unix nice level`: https://en.wikipedia.org/wiki/Nice_(Unix)
.. _`issues section`: https://github.com/gaia-pipeline/gaia/issues
.. _`Go installed`: https://golang.org/doc/install
.. _`nodeJS`: https://nodejs.org/
.. _`go-example repo`: https://github.com/gaia-pipeline/go-example
.. _`gitter`: https://gitter.im/gaia-pipeline

.. |build-status| image:: https://circleci.com/gh/gaia-pipeline/gaia/tree/master.svg?style=svg&circle-token=c0e15edfb08f8076076cbbb55558af6cfecb89b8
    :alt: Build Status
    :scale: 100%
    :target: https://circleci.com/gh/gaia-pipeline/gaia/tree/master

.. |go-report| image:: https://goreportcard.com/badge/github.com/gaia-pipeline/gaia
    :alt: Go Report Card
    :target: https://goreportcard.com/report/github.com/gaia-pipeline/gaia

.. |go-doc| image:: https://godoc.org/github.com/gaia-pipeline/gaia?status.svg
    :alt: GoDoc
    :target: https://godoc.org/github.com/gaia-pipeline/gaia

.. |apache2| image:: https://img.shields.io/badge/license-Apache-blue.svg
    :alt: Apache licensed
    :target: https://github.com/gaia-pipeline/gaia/blob/master/LICENSE

.. |chat| image:: https://img.shields.io/gitter/room/nwjs/nw.js.svg   
    :alt: Gitter
    :target: https://gitter.im/gaia-pipeline

.. |sh-login| image:: https://cdn.rawgit.com/michelvocks/6868118d0da06a422e69e453497eb30d/raw/142a2969c4d27d4135ef8f96213bb166009fda1e/login.png
    :alt: gaia login screenshot
    :width: 650px

.. |sh-overview| image:: https://cdn.rawgit.com/michelvocks/6868118d0da06a422e69e453497eb30d/raw/142a2969c4d27d4135ef8f96213bb166009fda1e/overview.png
    :alt: gaia overview screenshot
    :width: 650px

.. |sh-create-pipeline| image:: https://cdn.rawgit.com/michelvocks/6868118d0da06a422e69e453497eb30d/raw/ea6d76ad0cd9b30820149fb2e0fbdcdb101e1484/create_pipeline.png
    :alt: gaia create pipeline screenshot
    :width: 650px

.. |sh-create-pipeline-history| image:: https://cdn.rawgit.com/michelvocks/6868118d0da06a422e69e453497eb30d/raw/142a2969c4d27d4135ef8f96213bb166009fda1e/create_pipeline_history.png
    :alt: gaia create pipeline history screenshot
    :width: 650px
    
.. |sh-pipeline-detailed| image:: https://cdn.rawgit.com/michelvocks/6868118d0da06a422e69e453497eb30d/raw/51b4d6cbc3d86b1fe9531250db5456595423d9ec/pipeline_detailed.png
    :alt: gaia pipeline detailed screenshot
    :width: 650px
    
.. |sh-pipeline-logs| image:: https://cdn.rawgit.com/michelvocks/6868118d0da06a422e69e453497eb30d/raw/51b4d6cbc3d86b1fe9531250db5456595423d9ec/pipeline_logs.png
    :alt: gaia pipeline logs screenshot
    :width: 650px

.. |sh-settings| image:: https://cdn.rawgit.com/michelvocks/6868118d0da06a422e69e453497eb30d/raw/142a2969c4d27d4135ef8f96213bb166009fda1e/settings.png
    :alt: gaia settings screenshot
    :width: 650px
