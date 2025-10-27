---
date: '2025-10-27T16:12:49+08:00'
draft: false
title: '清除github幽灵通知 火狐浏览器'
---

[Notifications](https://github.com/notifications)这个网址里有某个仓库的通知无法被选中，
于是搜索并[山风阁](从https://blog.sunlan.me/2025/10/13/清除GitHub幽灵通知/)拿到了两行JavaScript，在FireFox上运行之后清掉了。
文章中描述的是Chrome的用法，之前我因为Feedbro被禁用搬到FireFox，所以这里重点说明如何在FireFox里使用。
把JavaScript片段贴到这里防止找不到：
```JavaScript
document.querySelector('.js-notifications-mark-all-actions').removeAttribute('hidden');
document.querySelector('.js-notifications-mark-all-actions form[action="/notifications/beta/archive"] button').removeAttribute('disabled');
```
1. 来到github的通知页面，在通知页面左侧，选中当前正在发送通知的幽灵仓库，这使得github的通知搜索栏里显示`repo:`开头的标签
2. 点击“全选”（Select all）
2. 按F12打开或从“工具”选项打开“开发者工具”
3. 点击控制台，按Ctrl+B（或Command+B）打开切换到多行模式
4. 尝试点击左边的栏目并粘贴代码，输入`allow pasting`确认粘贴代码
5. 点击代码上方、控制台下方的运行按钮运行代码，这使得“全选”（Select all）之后的操作面板显示出来
6. 点击Done，完成清除，等待刷新响应或手动刷新网页，该幽灵仓库应当不再产生通知圆点

