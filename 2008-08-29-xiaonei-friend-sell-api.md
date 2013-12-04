---
layout: article
title: 校内网好友买卖的API
date: 2008-08-29 16:05
category: 好友买卖外挂
excerpt:
  好友买卖是最近在校内网上比较火的小游戏，既然游戏的服务器是程序在控制，
  那为什么非要由人去玩呢？此文记录了我发现好友买卖API的过程（虽然它没有公开），
  并借助这个接口实现了自动玩好友买卖的程序。
---

真是不容易呀，连续好几个不眠的夜晚总算没有白费，我终于实现了自动买卖好友的功能，可以在频繁的鼠标操作中解放出来了！

起初我在Google里搜索好友买卖的漏洞时，出来的结果都是以前的GET和POST漏洞，现在这些漏洞都已经不能再利用了。

我用的系统是fodora 6，自带了一个命令行下的浏览器——curl，于是我就琢磨：既然它是一个浏览器，那也应该能提交让宠物挖煤、卖花的请求吧。于是先阅读一下curl的manual，然后一步一步实现自动化。

# 登入校内网

要玩好友买卖，首先得登入校内网，所以先下载校内网首页：

```bash
curl -o index.html http://xiaonei.com/
```

然后以“form”为关键字查找登入的表单，定位到如下代码：

```html
<form id="loginForm" class="login-form"
      method="post" action="http://login.xiaonei.com/Login.do" >
<p class="top">
<label for="email">登录邮箱：</label>
<input type="text" name="email" rule="blank#email"
       error="请输入正确的email地址" class="input-text"
       value="" id="email" tabindex="1" />
</p>
<p>
<label for="password">密码：</label>
<input type="password" id="password" name="password"
       rule="blank" error="请输入密码" class="input-text" tabindex="2"  />
```

仔细研究这段代码，发现表单是把email和password提交给 `http://login.xiaonei.com/Login.do`，用curl模拟登入：

```bash
curl -O -D cookie -d "email=username" -d "password=pwd" \
     http://login.xiaonei.com/Login.do
```

其中，-D是指定cookie的保存地址；-d是将后面的信息以post方式提交信息。打开下载来的Login.do，内容如下：

```html
The URL has moved <a href="http://home.xiaonei.com/Home.do">here</a>
```

页面转到了个人主页上，说明登入成功了！此时cookie文件中就保存着session-id等登入信息。

# 好友买卖

现在访问好友买卖的首页了：

```bash
curl -O -b cookie http://friendsell.xiaonei.com/friendsell/home.do
```

其中参数 -b 用于指定使用 cookie 文件。从home.do中定位到对好友处理的代码段：

```html
<li id="release_f" onclick="releasePet(222729759,this)"><a
        href="javascript:void(0)">抛弃她</a></li>
<li id="discount_f" onclick="discountPet(222729759,this)"><a
        href="javascript:void(0)">打折处理</a></li>
<li id="torment_f" rel="222729759" onclick="abuse(222729759,this)"><a
        href="javascript:void(0)">折腾她</a></li>
<li id="comfort_f" rel="222729759" onclick="comfortPet(222729759,this)"><a
        href="javascript:void(0)">安抚她</a></li>
<li id="nickname_f" rel="222729759" onclick="nickname(222729759,this)"><a
        href="javascript:void(0)">起昵称</a></li>
```

我们要做的当然是折磨他咯！不过点击后是触发一个js函数，文件里找不到abuse这个函数，看来还有另外一个js文件。我在顶部看到了 `http://xnimg.cn/xnapp/frs/frs.js?ver=63283` 这个链接，直接在浏览器里打开看看。就能看到所有的js函数。原来作者还预留了很多功能哈！找到abuse函数内容如下：

```javascript
function abuse(id, obj) {
    XN.DOM.disable(0);
    var myurl = "/friendsell/abuseAjax.do";
    var pars = "id=" + id;
    var myAjax = new Ajax.Request(
            myurl,
    {
        method: 'post',
        parameters: pars,
        onComplete: function(r) {
            XN.DOM.enable();
            displayAbus(r, XN.Element.realLeft(obj), XN.Element.realTop(obj));
        },
        onFailure: function() {
        }
    });
}
```

原来是把id提交给abuseAjax.do来处理。用curl提交：

```bash
curl -O -b cookie -d "id=friend_id" \
     http://friendsell.xiaonei.com/friendsell/abuseAjax.do
```

获得以下JSON信息（格式经过整理）：

```javascript
[{
    content: "<div class=\"dialog_summary\">
  <div style=\"margin: 10px auto; width: 300px; text-align: center;\">
    <img width=\"100\"
src=\"http://hd34.xiaonei.com/photos/hd34/20080510/12/43/head_2695h107.jpg\"/>
    <div class=\"m10_0\">
      <table>
        <tbody>
          <tr>
            <td class=\"graytext w100\">
              你要让***做什么呢？
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <hr size=\"1\" class=\"mb10\"/>
    <div class=\"cmplist mb10\">
      <ul>
        <form id='abuse_form' method='post' action='/friendsell/abuse.do'>
          <li>
            <input type=\"radio\" name=\"paintype\"
                   checked=\"check\" id=\"paintype\" value=\"1\"/>
            在冰冷小黑屋关一天
            <font color=\"#666666\">
              (消费￥1000)
            </font>
          </li>
          <li>
            <input type=\"radio\" name=\"paintype\"
                   id=\"paintype\" value=\"2\"/>
            痛扁她一顿
            <font color=\"#666666\">
              (消费￥100)
            </font>
          </li>
          <li>
            <input type=\"radio\" name=\"paintype\"
                   id=\"paintype\" value=\"3\"/>
            饿一天不给饭吃
          </li>
          <li>
            <input type=\"radio\" name=\"paintype\"
                   id=\"paintype\" value=\"11\"/>
            去看芙蓉姐姐演唱会
          </li>
          <li>
            <input type=\"radio\" name=\"paintype\"
                   id=\"paintype\" value=\"8\"/>
            去卖花
          </li>
          <li>
            <input type=\"radio\" name=\"paintype\"
                   id=\"paintype\" value=\"9\"/>
            跳草裙舞
          </li>
          <li>
            <input type=\"radio\" name=\"paintype\"
                   id=\"paintype\" value=\"10\"/>
            半夜闹铃
          </li>
          <input type='hidden' name='id' value='friend's id'>
        <\form>
      <\ul>
    <\div>
  <\div>
<\div>",
    id: "friend's id",
    name: "***"
}]
```

内容很明显是弹出的窗口。接下来就是把paintype和好友的id以post形式提交给abuse.do。但到了一步我卡住了，我还是依然提交：

```bash
curl -O -b cookie -d "paintype=8" -d "id=***" \
     http://friendsell.xiaonei.com/friendsell/abuse.do
```

但却没有返回成功的信息。我直接在firefox里访问“`http://friendsell.xiaonei.com/friendsell/abuse.do?paintype=8&id=***`”，结果弹出一个窗口：“不要修改链接哦^_^”。看来是作者在修补以前的漏洞时做的修改。

这让我感觉有点奇怪，莫非还有什么隐藏的信息要提交吗？为了验证自己的想法，我用winSock来抓包，看看firefox提交和手工get提交的区别。结果，除了提交方式，貌似看不出其他的区别！正当我绝望的时候，突然灵光一闪，是不是abuse.do对前面的页面abuseAjax.do有依赖？于是我运行：

```bash
curl -O -b cookie -d "paintype=8" -d "id=***" \
     -e http://friendsell.xiaonei.com/friendsell/abuseAjax.do \
     http://friendsell.xiaonei.com/friendsell/abuse.do
```

成功了！然后我写了个shell，对我所有的宠物进行提交，结果全部成功！

# 好友买卖小漏洞

起初我在想，男生没有买花的选项，不晓得行不行；但尝试提交看看，结果却成功了！如下图，其中红色圈的是男生，却也能买花。同样，女生也能挖媒！

{% img 1.png %}

这不算是什么大漏洞，但却方便了我写脚本，无需判断宠物的性别，统一去挖媒即可。

## 自动化

先总结一下需要自动化的过程：

1. 登入校内，保存cookie
1. 下载好友买卖首页到本地
1. 提取自己的宠物的id
1. 逐一安排宠物去打工

现在要做的就是把这些过程写成一个shell脚本，代码如下：

```bash
#!/bin/bash

if [ $# -eq 2 ]; then
    email=$1
    password=$2
else
    echo "usage: $0 email password"
    exit 1
fi

# Login
curl -O -D cookie \
     -d "email=${email}" \
     -d "password=${password}" \
     "http://login.xiaonei.com/Login.do"

# Get pet list.
curl -O -b cookie "http://friendsell.xiaonei.com/friendsell/home.do"

URL='http://friendsell.xiaonei.com/friendsell/'
sed -n -e '/ \{20\}/{s/[^0-9]\+//g;p}' home.do | while read id; do
    curl -O -b cookie \
         -d "paintype=5" \
         -d "id=${id}" \
         -e "${URL}abuseAjax.do" \
         "${URL}abuse.do"
done

# Clean.
rm -f *.do cookie
```

最后设定一个cron任务，指定每一小时自动运行一次。

```bash
crontab -e
0 */1 * * * ~/friendsell/pet email password
```

用同样的方法，还能实现自动补充好友，确保好友总数一直持续在12只。
