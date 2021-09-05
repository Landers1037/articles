---
title: 百度网盘直链下载js脚本
name: baidu-js-download
date: 2018-03-09
id: 0
tags: [百度网盘,脚本]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


 

```js
var CURL, DURL

try {
  CURL = $context.safari.items.location.href
} catch (e) {
  CURL = $clipboard.link
} finally {
  if (/pan.baidu.com/g.test(CURL)) {
    $http.get({
      url: CURL,
      header: {
        "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 10_1_1 like Mac OS X; zh-CN) AppleWebKit/537.51.1 (KHTML, like Gecko) Mobile/14B100 UCBrowser/10.7.5.650 Mobile"
      },
      handler: function(resp) {
        var data = resp.data.match(/(window\.yunData = ).*?(?=;)/g)
        data = JSON.parse(data[0].split("=")[1])
        list = data.file_list[0]
        Name = list.path.split("/").pop()
        DURL = "https://pan.baidu.com/share/download?bdstoken=" + data.bdstoken + "&web=5&app_id=" + list.app_id + "&logid=" + data.sharesuk + "=&channel=chunlei&clienttype=5&uk=" + data.uk + "&shareid=" + data.shareid + "&fid_list=%5B" + list.fs_id + "%5D&sign=" + data.downloadsign + "&timestamp=" + data.timestamp + "&r=0.7219390214898602"
        $http.get({
          url: DURL,
          handler: function(resp) {
            var data = resp.data
            if (!data.errno) {
              $ui.toast("直链已复制到剪贴板...")
              $clipboard.text = data.dlink
              $ui.menu({
                items: ["链接已复制", "直接下载", "迅雷下载", "直接退出"],
                handler: function(title, idx) {
                  switch (idx) {
                    case 1:
                      $http.download({
                        url: data.link,
                        handler: function(resp) {
                          var file = resp.data
                          $share.sheet([Name, file])
                        }
                      })
                      break;
                    case 2:
                      $app.openURL("thunder://" + data.dlink)
                      break;
                    default:
                      $app.close()
                      $context.close()
                      break;
                  }
                }
              })
            } else {
              $ui.alert("直链获取失败...")
              $delay(1, function() {
                $app.close();
                $context.close();
              })
            }
          }
        })
      }
    })
  } else {
    $ui.alert("不是有效的百度网盘链接...")
    $delay(1, function() {
      $app.close();
      $context.close();
    })
  }
}
```

        ios用户使用jsbox运行脚本提取直链，使用safari或者迅雷下载