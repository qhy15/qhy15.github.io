---
layout: post
title:  "解决jekyll因为categories含有中文而崩溃的问题"
date:   2014-07-09 16:00:00
categories: 	"使用jekyll建立个人网站"
tags: [jekyll]
---
###背景  
jekyll 版本	2.0.3	
之前_post文件夹下的文章，均是按照模板新写的，只在内容和题目部分、Tag部分涉及到中文，均支持的很好。
###问题描述

①在.md中加入  
	
	categories:	"中文"
	
出现如下错误：

	jekyll 2.0.3 | Error:  invalid byte sequence in US-ASCII
	
②在.md的名称中涉及到中文，例如

	2010-05-10-灌水.md
	
也会出现①中相同错误。
###思考

####1、定位错误

build运行时，打印详细错误：

	Generating... 
	/Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll.rb:121:in `gsub!': invalid byte sequence in US-ASCII (ArgumentError)
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll.rb:121:in `sanitized_path'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/post.rb:276:in `destination'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/cleaner.rb:43:in `block in new_files'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/site.rb:420:in `block (2 levels) in each_site_file'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/site.rb:419:in `each'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/site.rb:419:in `block in each_site_file'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/site.rb:418:in `each'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/site.rb:418:in `each_site_file'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/cleaner.rb:43:in `new_files'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/cleaner.rb:24:in `obsolete_files'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/cleaner.rb:15:in `cleanup!'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/site.rb:255:in `cleanup'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/site.rb:44:in `process'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/command.rb:43:in `process_site'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/commands/build.rb:46:in `build'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/commands/build.rb:30:in `process'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll/commands/serve.rb:23:in `block (2 levels) in init_with_program'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/mercenary-0.3.3/lib/mercenary/command.rb:220:in `call'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/mercenary-0.3.3/lib/mercenary/command.rb:220:in `block in execute'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/mercenary-0.3.3/lib/mercenary/command.rb:220:in `each'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/mercenary-0.3.3/lib/mercenary/command.rb:220:in `execute'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/mercenary-0.3.3/lib/mercenary/program.rb:35:in `go'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/mercenary-0.3.3/lib/mercenary.rb:22:in `program'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/bin/jekyll:18:in `<top (required)>'
	from /Users/qhy15/.rvm/rubies/ruby-2.1.1/bin/jekyll:23:in `load'
	from /Users/qhy15/.rvm/rubies/ruby-2.1.1/bin/jekyll:23:in `<main>'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1/bin/ruby_executable_hooks:15:in `eval'
	from /Users/qhy15/.rvm/gems/ruby-2.1.1/bin/ruby_executable_hooks:15:in `<main>'

然后定位到出现错误的位置:

<font color='red'>
	/Users/qhy15/.rvm/gems/ruby-2.1.1@global/gems/jekyll-2.0.3/lib/jekyll.rb
	</font>

	119	def self.sanitized_path(base_directory, questionable_path)
    120 clean_path = File.expand_path(questionable_path, fs_root)
	121 clean_path.gsub!(/\A\w\:\//, '/')
	
即，造成错误出现的语句为	
	<font color='red'>
	clean_path.gsub!(/\A\w\:\//, '/')
	</font>
	
####2.错误原因分析

首先，提示信息`invalid byte sequence in US-ASCII`说明了gsub的参数出现了非ASCII字符，这与问题描述中提到的新加入的中文字符对应。所以首先考虑是不是项目的编码方式设置错误。

但是，jekyll的[官方文档](http://jekyllrb.com/docs/configuration/)提到2.0.0之后的版本，默认编码为utf-8；页面中的中文与TAG字段的中文都能支持。  
除此之外，`_site`文件夹下的文章对应的html均已经生成，也能排除编码方式设置错误的原因。

只有根据  
`jekyll.rb:121:in 'gsub!': invalid byte sequence in US-ASCII (ArgumentError)`  
为关键词进行搜索。

在第一个[搜索结果页面](http://github.com/jekyll/jekyll/issues/2379)中，有人出现了一样的问题。  

[页面](http://github.com/jekyll/jekyll/issues/2379)中，[penibelst](http://github.com/penibelst)首先给出他的质疑：
	
>The Wuxia post contains Chinese characters in front-matter’s tags and category. I don’t think it’s allowed here.

他认为中国字符不能出现在tag和category中。  

但是我遇到的问题是仅category中的中文字符有问题，tag中的中文字符是能够支持的。我猜想，tag和category的处理方式应该是一样的，唯一的不同是，根据category的名字在`_site文件夹`下生成了对应的文件夹。  
页面中也马上有用户质疑了这个`tag和category中不能有非ASCII码字符`的想法。  

然后，[albertogg](http://github.com/albertogg)给出了解决方法，把出错的一行改为如下就能解决问题。
		
	clean_path.force_encoding('UTF-8').gsub!(/\A\w\:\//, '/')  

接着，[fabianrbz](http://github.com/fabianrbz)（jekyll的一个contributor）给出了可能造成这个问题的原因：
	
>he issue was introduced when we started using `URL` instead of `CGI` this is the commit that introduced [this](http://github.com/jekyll/jekyll/commit/eebb6414bf9dbcb3e02ed11b22cde3e6f96b7f4f) issue.
<br><br>
>I'll investigate a bit more the differences between the two.
<br>
We can also use `addressable/uri`  

[fabianrbz](http://github.com/fabianrbz)说，他们在使用`URL`代替`CGI`的时候，发生了[url escape](http://github.com/jekyll/jekyll/commit/eebb6414bf9dbcb3e02ed11b22cde3e6f96b7f4f)——
如果生成的html文件名含有空格时，转换后将指向一个不存在的html文件。  
例如，post
<font color=red>
“2014-01-02-foo bar.md”
</font>
时，期待的文件名为
<font color=red>
“/2014/01/02/foo%20bar.html”
</font>
，而实际生成的文件名为
<font color=red>
“/2014/01/02/foo+bar.html”
</font>。

最后，给出解决方案的[albertogg](http://github.com/albertogg)做了进一步的[研究](http2://github.com/jekyll/jekyll/pull/2420)，并最终把这个bug[修复](https://github.com/jekyll/jekyll/pull/2420)了。  

总结上述内容，简单来说，造成tag中不支持中文的问题是`jekyll 2.0.3`中的一个bug。  
>What Jekyll is doing right now: It gets a UTF-8 string, when it gets escaped, the string is converted to ASCII, but when it is unescaped it remains ASCII and this is when the problem occurs. 

`jekyll 2.0.3`在处理url时，`lib/jekyll/url.rb`中调用escape把UTF-8字符串转换为ASCII格式，但是unescape的时候字符串保持为ASCII格式并没有恢复为UTF-8字符串，导致了bug出现。  
相关bug代码  

	 URI.escape(path, /[^a-zA-Z\d\-._~!$&\'()*+,;=:@\/]/)
     ...
     URI.unescape(path)

修复bug后的相关代码为  
	 
	 URI.escape(path, /[^a-zA-Z\d\-._~!$&\'()*+,;=:@\/]/).encode('utf-8')
	 ...
	 URI.unescape(path.encode('utf-8'))
	 
###问题解决办法
####1、jekyll 2.0.3
修改`jekyll-2.0.3/lib/jekyll.rb`文件中

	clean_path.gsub!(/\A\w\:\//, '/')
为	
	
	clean_path.force_encoding('UTF-8').gsub!(/\A\w\:\//, '/')  

####2、更新jekyll部分代码至最新
从[github](http://github.com/jekyll/jekyll)同步最新的代码。