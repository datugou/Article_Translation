# 如何在互联网上找到任何东西
_作者：[**Alec Barrett-Wilsdon**](https://www.alec.fyi/pages/about)_  
_原文链接：<https://www.alec.fyi/dorking-how-to-find-anything-on-the-internet.html#seo>_  
_2020 年 4 月 4 日_  
`Google` `Docking`

---
软件工程师长期以来一直开玩笑说，他们的工作有一大部分内容是简单的在 Google 上搜东西。
现在你可以做同样的事情，并且是免费的。
下面，我将介绍 dorking 技术，即利用搜索引擎查找非常具体的数据。
对于每个例子，你可以直接粘贴到 Google 中查看结果。

### 目录：
- 网页
- 电子邮件
- 文档
- 搜索优化
- 优惠券！
- 秘密
- 操作符回顾


## 网页
受这次与 Gumroad 的 CEO Sahil Lavingia 在 [Twitter 上交流](https://twitter.com/Contextify1/status/1255202725551202304)的启发，接下来的几个例子将涵盖 Gumroad 和 Sahil。

在网站中找到特定的页面（例如：DynamoDB 电子书）
```
site:gumroad.com dynamodb
```

找到特定的页面：必须在标题文本中包含一个短语
```
allintitle:"support this" site:gumroad.com
```

找同类网站
```
related:gumroad.com
```

你可以将操作符连锁在一起（例如：在 URL 中寻找包含 security 或 bug-bounty 的 bug 悬赏内容）
```
(inurl:security OR inurl:bug-bounty OR site:hackerone.com) + "gumroad"
```

你可以将搜索范围限制在特定的顶级域命（如：你想找到某个教师名单）
```
site:.edu filetype:xls inurl:"email.xls"
```

## 电子邮件
查找 Gmail 账户
```
sahil lavingia "@gmail.com"
```

找到工作账户（你需要先找到他们的域名）
```
sahil lavingia "@gumroad.com"
```

没有找到你要找的东西？试着猜测一下邮件的格式（试着去这个[网站](https://www.email-format.com/d/shopify.com/#)，搜索域名，然后点击**识别名称格式**）
```
"s.lavingia" "@" ".com"
```

你可以随时找到每一个有邮件的页面
```
site:gumroad.com intext:"@gumroad.com"
```

找到你所在网页上的每一封邮件。
最重要的是，它适用于所有网站。
使用 Chrome 浏览器的开发者工具（DevTools）将它注入到网站中（[更多信息](https://www.alec.fyi/how-to-scrape.html)）。

``` js
var elems = document.body.getElementsByTagName("*")。
var re = new RegExp("(^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$)");
for (var i = 0; i < elems.length; i++) { ．
    if (re.test(elems[i].innerHTML)) {
        console.log(elems[i].innerHTML);
    }
}
```

这将记录每一封找到的邮件，而不需要你扫描整个页面。

最后，你可以在 Gmail 的一个新的撰写窗口中悬停在你发现或猜测的邮件上，来验证它是否真实。

<div align=center><img src="https://www.alec.fyi/images/dorking-1.png">   <img src="https://www.alec.fyi/images/dorking-2.png"></div>

如果你仔细观察，你会发现聊天和视频通话选项在无效邮件上是灰色的。

## 文档
查找电子表格
```
filetype:csv OR filetype:xlsx OR filetype:xls OR filetype:xltx OR filetype:xlt OR inurl:airtable.com/universe/
```

查找 Google 文档和 Google 表格
```
site:docs.google.com "gumroad"
```

查找竞争对手 LOGO 的位置（例如：合作伙伴或客户的网站）
```
"Gumroad Logo.png"
```

查找竞争对手的销售建议和白皮书
```
site:intercom.com (filetype:pdf OR filetype:ppt)
```

查找有关竞争对手的案例研究
```
inurl:hubspot-case-study -site:http://hubspot.com
```

## 搜索优化
在锚文本中找到含有特定关键词的网站
```
inanchor:"cyber security"
```

找到题目含有特定关键词的博客文章
```
inposttitle:"diy slime"
```

查找反向链接（例如：链接到某篇博客文章的其他网站）。注意：链接操作符现在已经过时了
```
intext:intercom.com/intercom-api-reference/reference
```

查找时在关键词中使用通配符
```
* design tools
```

查找带有给定条件的公司
```
intext:"Powered by Intercom" -site:intercom.com
```

## 优惠券！
搜索网站本身
```
site:curology.com ("coupon" | "referral code" | "affiliate code" | "discount code" | "VIP")
```

在 twitter 上搜一搜
```
site:twitter.com + "meundies" + ("coupon" | "referral code" | "affiliate code" | "discount code" | "VIP")
```

再去 Mailchimp 邮件上试试 
```
site:campaign-archive.com + "blueapron" + ("coupon" | "referral code" | "affiliate code" | "discount code" | "VIP")
```

## 秘密
网络安全专家使用 dorking 作为众多工具中的一种，来寻找公司的潜在漏洞。
出于对其可能被滥用的担忧，本文将不涉及任何此类查询。

## 操作符回顾
操作符是搜索查询的组成部分，可以缩小搜索结果的范围。
你可以在一个查询中组合任意多的操作符。
下面是最常用的操作符。

|操作|符描述|
|-|-|
|"phrase"| " 括住**搜索词**，结果必须包括**搜索词**
|-phrase| - 加**搜索词**，排除带有**搜索词**的结果
|phrase1 AND phrase2|用 AND 连接搜索词
|phrase1 OR phrase2|用 OR 连接搜索词
|site:example.com|site:**域名**，结果必须在域名 example.com 上
|filetype:jpg|filetype:**格式**，结果必须是 .jpg 类型的

AND/OR 逻辑词可以用来组合不同的查询
```
"phrase1" OR "phrase 2" AND "phrase3"
# 上面的这条搜索，相当于下面这两个搜索的集合
>> "phrase1"
>> "phrase 2" AND "phrase3"
```

---
[返回目录](https://github.com/datugou/Article_Translation)
