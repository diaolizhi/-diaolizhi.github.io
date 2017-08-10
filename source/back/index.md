---
title: 关于

heart: true
---


[搭建教程](http://www.wuxubj.cn/2016/08/Hexo-nexT-build-personal-blog/)
[Next 使用文档](http://theme-next.iissnan.com/getting-started.html#install-next-theme)
[Hexo 文档](https://hexo.io/zh-cn/docs/tag-plugins.html)
[Next 使用说明](https://binglumeng.github.io/2017/03/21/Hexo%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)
[Next 主题个性化配置](https://segmentfault.com/a/1190000009544924#articleHeader21)
[Hexo 你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)



---
安装 Git
安装 Node.js https://nodejs.org/en/download/
命令行安装 
	npm install hexo-cli -g
	npm install hexo --save
创建 Hexo 文件夹
	hexo init <folder>
	cd <folder>
	npm install
安装插件
	npm install hexo-generator-index --save
	npm install hexo-generator-archive --save
	npm install hexo-generator-category --save
	npm install hexo-generator-tag --save
	npm install hexo-server --save
	npm install hexo-deployer-git --save
	npm install hexo-deployer-heroku --save
	npm install hexo-deployer-rsync --save
	npm install hexo-deployer-openshift --save
	npm install hexo-renderer-marked@0.2 --save
	npm install hexo-renderer-stylus@0.2 --save
	npm install hexo-generator-feed@1 --save
	npm install hexo-generator-sitemap@1 --save
启动服务
	hexo server
安装 next 主题，在博客目录下
	git clone https://github.com/iissnan/hexo-theme-next themes/next
不用上面这步了，直接去 clone 别人的项目即可


------------------------------
新建项目 github.io 结尾
git clone git@github.com:diaolizhi/diaolizhi.github.io.git
新建一个文件，然后 commit 
新建分支，切换分支，粘贴文件
npm install
在配置文件里面修改：
	deploy:
		type: git
		repository: git@github.com:diaolizhi/diaolizhi.github.io.git
		branch: master
提交分支里面的文件
生成 github 网站上网页文件
	hexo d -g
将 master 分支的内容同步到本地（其实没必要）
---