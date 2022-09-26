---
layout: mypost
title: 关于我
---

*#音量注意：没有调节音量的功能，播放前请注意设备音量*

<iframe src="//music.163.com/outchain/player?type=2&id=1451141913&auto=1&height=66" frameborder="0" width="100%" height="86px" ></iframe>

> Hello 旅行者，欢迎访问……

## 关于本博客

> <span id="sitetime"  style="color: #00808c; text-align: center; padding: 15px 0; font-size: 14px;" ></span>

*搭建与维护本博客的全部工作都是在工作日20:00至24:00之间及休息日完成的，期间没有一条鱼被摸，请放心阅读(•̀ᴗ•́)و ̑̑

*推荐使用电脑浏览器访问，配合页面上的音乐，以获得最佳阅读体验👏

计划是以后有时间了就会写点东西进来，包括学习笔记/书签备忘/生活趣事等等，预期更新频率是两周一篇~~（快停止你的挖坑行为）~~什么时候写够十篇文章我就公开👏

该静态博客基于 Gitee Pages 服务 + jekyll 框架搭建，图床采用 Github + jsDelivr + PicGo + Typora 的方案管理，以保证访问速度~~（其实主要还是因为都能白嫖）~~

特别感谢[Tmize](https://github.com/TMaize/tmaize-blog)大佬提供的 jekyll 主题，清爽得不得鸟

另外欢迎添加友链，在[留言板](chat.html)留言即可

## 关于我感兴趣的东西

计算机，剪辑，摄影，ACG，LOL，足球，猫……

多而不精，广而不深，技能树点歪了属于是orz

## 关于我的其他信息

暂时不知道写什么了，想起来了再补充8

## 找到我

- [哔哩哔哩](https://space.bilibili.com/32229855)：鸽了，下次剪amv有生之年(￣3￣)
- [微&nbsp;&nbsp;博](https://weibo.com/u/6048615002)：主要发自家的大猫和小猫，有两个专门的#话题#( ´∀`)
- [微信公众号](https://mp.weixin.qq.com/s/g9xSqxS42DeXbohHFgqMoA)：只有这一篇文章，以后应该也不会更新( ﾟ∀ﾟ)
- [留言板](chat.html)：欢迎来[留言互动](chat.html)~(`・ω・´)
- [Email](http://mail.qq.com/cgi-bin/qm_share?t=qm_mailme&email=27izvraEuLqvm6qq9bi0tg): 点击<a target="_blank" href="http://mail.qq.com/cgi-bin/qm_share?t=qm_mailme&email=27izvraEuLqvm6qq9bi0tg" style="text-decoration:none;">给我写信</a>~(ゝ∀･)

## 知识共享许可协议

- **博客作者：**Chemcat

- **博客链接：**[https://chemcat.gitee.io/](https://chemcat.gitee.io/)

- **版权声明：**本博客所有文章除特别声明外，均采用 `CC BY-NC-SA 4.0` 许可协议。

<script>
    function siteTime(){
        window.setTimeout("siteTime()", 1000);
        var seconds = 1000;
        var minutes = seconds * 60;
        var hours = minutes * 60;
        var days = hours * 24;
        var years = days * 365;
        var today = new Date();
        var todayYear = today.getFullYear();
        var todayMonth = today.getMonth()+1;
        var todayDate = today.getDate();
        var todayHour = today.getHours();
        var todayMinute = today.getMinutes();
        var todaySecond = today.getSeconds();
        /* Date.UTC() -- 返回date对象距世界标准时间(UTC)1970年1月1日午夜之间的毫秒数(时间戳)
        year - 作为date对象的年份，为4位年份值
        month - 0-11之间的整数，做为date对象的月份
        day - 1-31之间的整数，做为date对象的天数
        hours - 0(午夜24点)-23之间的整数，做为date对象的小时数
        minutes - 0-59之间的整数，做为date对象的分钟数
        seconds - 0-59之间的整数，做为date对象的秒数
        microseconds - 0-999之间的整数，做为date对象的毫秒数 */
        var t1 = Date.UTC(2021,12,05,00,00,00); //北京时间创建网站的时间
        var t2 = Date.UTC(todayYear,todayMonth,todayDate,todayHour,todayMinute,todaySecond);
        var diff = t2-t1;
        var diffYears = Math.floor(diff/years);
        var diffDays = Math.floor((diff/days)-diffYears*365);
        var diffHours = Math.floor((diff-(diffYears*365+diffDays)*days)/hours);
        var diffMinutes = Math.floor((diff-(diffYears*365+diffDays)*days-diffHours*hours)/minutes);
        var diffSeconds = Math.floor((diff-(diffYears*365+diffDays)*days-diffHours*hours-diffMinutes*minutes)/seconds);
        document.getElementById("sitetime").innerHTML="Chemcat Blog 搭建至今已稳定运行***"+diffDays+"天"+diffHours+"小时"+diffMinutes+"分钟"+diffSeconds+"秒***"; //+diffYears+"年"
    }
    siteTime();
</script>