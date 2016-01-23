title: hexo-quick-start
date: 2016-01-23 15:36:16
categories: Hexo
tags:
- Hexo
- Setting

---
Welcome to [Hexo](http://hexo.io/)! This is your very first post. Check [documentation](http://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](http://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Environment

```
	$cd //folder
	$hexo init
	$npm install
	$vi _config.yml
		deploy:
  		type: git
  		repo: https://github.com/ccSun/ccsun.github.io.git
		branch: master
	$hexo g
	$hexo d
```

## Error: Deployer not found: git

``` bash
$ npm install hexo-deployer-git --save
```

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](http://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](http://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](http://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](http://hexo.io/docs/deployment.html)
