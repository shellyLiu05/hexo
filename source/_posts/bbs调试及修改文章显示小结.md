---
title: bbs调试及修改文章显示小结
date: 2018-07-26 09:25:10
tags:
---

## 目的

1. bbs编辑完成发表时，会发现实际显示的样式和编辑时的样式有差别
2. 复制markdown文档到bbs编辑器中，发表后显示的效果与markdown中不一致，会存在代码段换行失效问题

主要针对这些问题进行bbs显示的优化。

## 调试方法

bbs采用的是weCenter, weCenter可以在配置文件中开启网站的debug模式，然后通过debug函数输出信息到页面的最下角以供调试

配置文件路径为：在bbs目录下的`system\config\system.php`文件中。

具体修改如下：

```php
<?php

$config['debug'] = true;	// 开启网站 Debug 模式
```

打开该配置后，会有一些系统的默认调试信息输出在页面最底部。

如果需要打印其他调试信息，使用```AWS_APP::debug_log()```函数进行打印，该函数的定义在文件`system\aws_app.inc.php`中,接收三个参数，模块类型，打印时间，打印信息。

```php
	/**
	 * 记录系统 Debug 事件
	 *
	 * 打开 debug 功能后相应事件会在页脚输出
	 *
	 * @access	public
	 * @param	string
	 * @param	string
	 * @param	string
	 */
	public static function debug_log($type, $expend_time, $message)
	{
		self::$_debug[$type][] = array(
			'expend_time' => $expend_time,
			'log_time' => microtime(true),
			'message' => $message
		);
	}
```

## 解决方法

对于粘贴过来的markdown文档，发表时，存储在数据库中的数据与复制到编辑框中的数据是一样的，只是显示时是将从数据库读出来的文章内容放在pre标签中显示出来，而在`static\css\default\common.css`中设置了：

```css
.markitup-box pre br, .markitup-box ul + br, .markitup-box ol + br {display: none;}
```

而markdown中的代码段是使用br来换行的，这个导致了markdown文档中的代码段换行无法显示。

由于pre标签显示markdown表格等也有问题，而编辑的时候使用的是ueditor工具，该工具对markdown文档的支持良好，最终决定将文章的pre显示方式换成使用编辑工具ueditor显示。

## 使用ueditor显示

ueditor使用方法：

```javascript
// 初始化编辑器，将id=wmd-input的标签内容作为ueditor的显示内容
var editor = UE.getEditor('wmd-input');
//对编辑器的操作最好在编辑器ready之后再做
editor.ready(function(){
  //TODO
})
```

使用配置文件对ueditor进行配置，配置文件为`ueditor.config.js`，里面列出了可配置项，可自由配置。

还可以使用命令对ueditor进行配置。如`editor.setDisabled()`是禁用ueditor的所有编辑功能，这在显示时需要设置。

### 具体步骤

**步骤1：**

显示的html文件是`views\default\article\index.tpl.htm`，

修改

```php
<?php echo $this->article_info['message']; ?>
```

为：

```php
<textarea class="wmd-input editor" id="wmd-input" rows="15" name="question_detail">
	<?php echo $this->article_info['message']; ?>
</textarea>
```

**步骤2：**

在`static\js\app`路径下增加`article.js`来调用ueditor编辑器。为了优化显示效果，在文章显示时，去掉了ueditor默认的边框以及增加了url跳转处理。

`article.js`代码

```javascript
$(function()
{
		// 初始化编辑器
		var editor = UE.getEditor('wmd-input');
		//对编辑器的操作最好在编辑器ready之后再做
		editor.ready(function(){
			editor.setDisabled();

			//去掉ueditor边框和编辑框
			$('#edui1').css('border-style', 'none');
			$('#edui1_toolbarbox').css('display', 'none');
			$('#edui1_bottombar').css('display', 'none');

			//将iframe中的超链接都在新标签页中打开
			var iframs = document.getElementsByTagName('iframe');
			for(var i = 0; i < iframs.length; i++) {
				var ifrm = iframs[i];
				var head;
				if (ifrm.contentWindow)
				{
					head = ifrm.contentWindow.document.head;
				}
				else
				{
					head = ifrm.contentDocument.document.head;
				}
				if (head) {
					var base = document.createElement('base');
					base.target = "_blank";
					head.appendChild(base);
				}
			}
		});
});
```

**步骤3**

在`app/article/main.php`中index_action()函数中增加，当在请求文章内容时，便会加载`article.js`脚本和`ueditor`库文件。

```php
public function index_action()
{
  if ($_GET['notification_id'])
  {
  	$this->model('notify')->read_notification($_GET['notification_id'], $this->user_id);
  }

  if (is_mobile())
  {
  	HTTP::redirect('/m/article/' . $_GET['id']);
  }

  if (! $article_info = $this->model('article')->get_article_info_by_id($_GET['id']))
  {
  	HTTP::error_404();
  }

 + TPL::import_js('js/app/article.js');
 + import_editor_static_files();

  if ($article_info['has_attach'])
  {
  	$article_info['attachs'] = $this->model('publish')->get_attach('article', $article_info['id'], 'min');

  	$article_info['attachs_ids'] = FORMAT::parse_attachs($article_info['message'], true);
  }
  ...
}
```

至此，使用ueditor来显示文章内容已经完成

### 小细节优化

在测试中发现，使用ueditor编辑器显示文章时，不会将很长的一行的代码自动换行，文章中的图片宽度会超过显示的宽度。

于是通过在`static\ueditor1_4_3_3\themes\iframe.css`增加自定义的CSS样式来实现显示的优化，如下：

`iframe.css`文件:

```css
/*可以在这里添加你自己的css*/
span, pre{
        white-space: pre-wrap
}
span{
        max-width: 100%
}
img{
        max-width:100%;
}
```

还需要在ueditor的配置文件中打开`iframeCssUrl`配置，如下：

```javascript
...
//如果自定义，最好给p标签如下的行高，要不输入中文时，会有跳动感
//,initialStyle:'p{line-height:1em}'//编辑器层级的基数,可以用来改变字体等

,iframeCssUrl: URL + '/themes/iframe.css' //给编辑区域的iframe引入一个css文件
...
```

另，在测试过程中发现输入长文章会被截断，原因是文章内容在MySQL中存储时，使用的数据类型是TEXT，最多支持64KB的数据量，改为MEDIUMTEXT后就解决了该问题。

