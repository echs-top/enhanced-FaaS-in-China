# enhanced-FaaS-in-China

Improve the access speed and stability in China of web pages hosted on either vercel or netlify by merely changing your CNAME record from the official one to one I made<br>
提升部署在vercel或netlify的网页在中国的访问速度和稳定性

## Usage

* 如果你的网站部署在`vercel`，则把cname记录改为：
  * `vercel-cname.xingpingcn.top`
* 如果你的网站部署在`netlify`，则把cname记录改为：
  * `netlify-cname.xingpingcn.top`
* 如果你的网站部署在`netlify`和`vercel`上，则把cname记录改为：
  * `verlify-cname.xingpingcn.top`
    * 使用此dns解析建议：先把cname记录改为官方提供的url，等`ssl/tls证书`生成之后再把cname记录改为`verlify-cname.xingpingcn.top`

### 可能存在的问题

1. 浙江、福建、河南的个别isp访问可能会失败（官方的cname也是同样的问题，或许是isp限制导致的）
1. 对于测速工具的选择，itdog.cn测出来的结果有点问题（会出现大片的红，原因未知），可以试试用boce.com、cesu.net之内的来测

## Why to use it

1. 如果在大陆访问，官方的anycast会将流量大概率路由到东南亚，路线压力很大，但是存在压力较小的美国或者欧洲的路线。
1. 官方的cname有时在平均速度上是很快的，但是缺乏稳定性，会出现好几个省份都访问不了的情况，又或者个别省份相应时间非常长
1. 由于存在被墙风险，如果使用单一的平台——例如vercel——则会存在全军覆没的情况，既国内所有地方都不能访问你的网站。

![vercel中午](img/vercel-noon.png)
vercel中午

## 测速对比

<details>
<summary>点击查看结果</summary>

![vercel-23点晚高峰](img/vercel-23utc8.png)
vercel-23点晚高峰
![vercel-23点晚高峰-官方](img/vercel-23utc8-auth.png)
vercel-23点晚高峰-官方
![netlify-23点晚高峰](img/netlify-23utc8.png)
netlify-23点晚高峰
![netlify-23点晚高峰-官方](img/netlify-23utc8-auth.png)
netlify-23点晚高峰-官方
![vercel中午](img/vercel-noon.png)
vercel中午
![vercel中午-官方](img/vercel-noon-auth.png)
vercel中午-官方
![netlify中午](img/netlify-noon.png)
netlify中午
![netlify中午-官方](img/netlify-noon-auth.png)
netlify中午-官方

</details>

## How it works

选取vercel和netlify的IP，定时测试速度，选取稳定且快的ip添加到域名的A记录。国内有三网优化，国外统一使用官方提供的A记录。

大概每一小时更新一次。 

### IP来源

* vercel
  * [vercel ip](https://gist.github.com/ChenYFan/fc2bd4ec1795766f2613b52ba123c0f8)
  * 官方`cname.vercel-dns.com.`的A记录
* netlify
  * 官方所提供的链接的A记录

## Q&A

**Q：和官方提供的cname有什么差别？**

A：

* 官方的cname有时在平均速度上是很快的，但是缺乏稳定性，会出现好几个省份都访问不了的情况，又或者个别省份相应时间非常长
* 而我的cname在平均速度上可能不是最快的，但平均响应速度尽量维持在1秒内，最长的响应时间控制在2秒内，而返回非200状态码的省份尽量少于等于2个

**Q：为什么分路线解析不准确？**<br>
A：我使用的是权威DNS服务器自带的路线解析，可能存在误判。如果你想要更加精准的分路线解析，可以自行选取其他DNS服务器——如dnspod——并添加[Netlify.json](https://raw.githubusercontent.com/xingpingcn/enhanced-FaaS-in-China/main/Netlify.json)或[Vercel.json](https://raw.githubusercontent.com/xingpingcn/enhanced-FaaS-in-China/main/Vercel.json)里的IP到A记录。或使用`NS1.COM`作为权威DNS服务器，并设置根据`ASN`进行路线解析。你可以看看我写的[ASN列表](https://github.com/xingpingcn/china-mainland-asn)。<br>

**Q：为什么设置了你的CNAME解析后网站不能访问？**<br>
A：

* 这大概率是使用了`verlify-cname.xingpingcn.top`导致的。需要先把CNAME记录改为官方提供的链接，等待SSL证书生成后再重新设置。这是由于该解析包含两个平台的IP，平台每次访问都会获得二者之一的IP，因而认为你在平台所填写的域名并不是你所拥有的。但是一旦生成证书后，证书就会缓存在平台上。
* netlify支持上传自己的证书。如果还是不行就申请一个能自动续期的证书。

**Q：为什么有的路线（如电信）的DNS A记录解析是官方提供的默认IP？**<br>
A：这是因为该路线的其他IP质量较差，所以暂时停止解析其路线，改用官方提供的默认IP。你可以通过同时将网站部署在`vercel`和`netlify`，把cname解析改为`verlify-cname.xingpingcn.top`，从而提高容错率。两个平台同一线路同时失效的概率要低许多。

**Q：为什么在json文件种有的路线是一个空列表？**<br>
A: 同上


## Custom

如果你想自定义，例如增加第三个平台，如`render`、`railway`等，则需要自己准备测速工具和一个域名，并重写`crawler.py`，在`platforms_to_update`新建一个`.py`文件，仿照文件夹内的其他文件重写`update()`方法，最后修改`config.py`文件相关配置。