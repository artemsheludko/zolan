TIPs

* 所有的terminal的命令都要在文件夹 /Users/kirasun/Documents/portfolioWebsite/portfolio2023.github.io 下运行， 切换到这个文件夹的命令是
`
cd /Users/kirasun/Documents/portfolioWebsite/portfolio2023.github.io
`

### 更改网站后 如何在本地展示？
文件修改完成以后，在上面那个目录下运行 

`
bundle exec jekyll serve
`

在输出的最后，复制server address后的内容，粘贴到浏览器就可以在本地浏览。在terminal（iterm）里按下 ctrl+c可以停止本地网站的运行。


### 在本地预览完，没有问题以后怎么发布到远程的网站?
* 在GitHub Desktop里，浏览到正确的仓库（Repo），默认应该就是对的。
* 先把要更改的文件，打上√，然后填上提交的Summary，然后点击 Commit to Master，这一步不会真的提交，只是确认更改。
* 点击上面的Push 按钮，会真正的提交到远程的仓库（Repo）。 接下来就可以在远程的网站预览更改了。

### 远程预览更改

* 更改提交以后会经过Build And Deploy的过程，在页面 https://github.com/kardplayer/portfolio2023.github.io/actions 可以看到进展。等图标绿了以后，就生效了。
* 你的远程网站地址是https://kardplayer.github.io/portfolio2023.github.io/ 它在Setting -> Pages 里也可以找到。


### 怎么添加 Post
具体的步骤和说明在 https://github.com/kardplayer/portfolio2023.github.io#posts下面