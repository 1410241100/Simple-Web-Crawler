# 验证码识别

基于C#.NET的简单网页爬虫，支持异步并发、设置代理、操作Cookie、Gzip页面加速。


### 主要特性

- 支持Gzip根据网页内容自动解压，加快爬虫载入速度；
- 支持异步并发抓取；
- 支持自动事件通知；
- 支持代理切换;
- 支持操作Cookies；


### 运行截图	

- 用爬虫抓取携程网的酒店数据，看看效果如何？
- 携程的酒店是按城市归类的，从每个城市又链接到了下属所有酒店，国内的城市大约有300多个，仅北京一个市的酒店数据就有9000多个，所以我要先抓下边这页面里的城市名称及城市URL地址。
![携程网城市列表](https://github.com/coldicelion/Simple-Web-Crawler/blob/master/Wesley.Crawler.SimpleCrawler/Images/1.%E6%90%BA%E7%A8%8B%E7%BD%91%E5%9F%8E%E5%B8%82%E5%88%97%E8%A1%A8.png?raw=true)

- 现在请出《正则表达式》，我写了个提取城市名称及URL的正则表达式，直接提取源代码中所有符合规则的数据：
- &lt;a[^&gt;]+href=""*(?&lt;href&gt;/hotel/[^&gt;\s]+)""\s*[^&gt;]*&gt;(?&lt;text&gt;(?!.*img).*?)&lt;/a&gt;
![使用正则表达式清洗数据](https://github.com/coldicelion/Simple-Web-Crawler/blob/master/Wesley.Crawler.SimpleCrawler/Images/3.%E4%BD%BF%E7%94%A8%E6%AD%A3%E5%88%99%E6%B8%85%E6%B4%97%E6%95%B0%E6%8D%AE.png?raw=true)

- 成功拿到了城市名称及城市URL地址，每个城市的URL中都包含了该城市的所有酒店，现在我们就根据城市URL抓取下属的酒店列表，就拿最后一个城市来试试吧，再次使用正则表达式清洗数据，请原谅我写出这无比糟糕的正则表达式：
- "&gt;&lt;a[^&gt;]+href="*(?&lt;href&gt;/hotel/[^&gt;\s]+)"\s*data-dopost[^&gt;]*&gt;&lt;span[^&gt;]+&gt;.*?&lt;/span&gt;(?&lt;text&gt;.*?)&lt;/a&gt;
![抓取城市下的酒店列表](https://github.com/coldicelion/Simple-Web-Crawler/blob/master/Wesley.Crawler.SimpleCrawler/Images/4.%E6%8A%93%E5%8F%96%E5%9F%8E%E5%B8%82%E4%B8%8B%E7%9A%84%E9%85%92%E5%BA%97%E5%88%97%E8%A1%A8.png?raw=true)


### 示例代码

        /// <summary>
        /// 抓取城市列表
        /// </summary>
        public static void CityCrawler() {
            
            var cityUrl = "http://hotels.ctrip.com/citylist";//定义爬虫入口URL
            var cityList = new List<City>();//定义泛型列表存放城市名称及对应的酒店URL
            var cityCrawler = new SimpleCrawler();//调用刚才写的爬虫程序
            cityCrawler.OnStart += (s, e) =>
            {
                Console.WriteLine("爬虫开始抓取地址：" + e.Uri.ToString());
            };
            cityCrawler.OnError += (s, e) =>
            {
                Console.WriteLine("爬虫抓取出现错误：" + e.Uri.ToString() + "，异常消息：" + e.Exception.Message);
            };
            cityCrawler.OnCompleted += (s, e) =>
            {
                //使用正则表达式清洗网页源代码中的数据
                var links = Regex.Matches(e.PageSource, @"<a[^>]+href=""*(?<href>/hotel/[^>\s]+)""\s*[^>]*>(?<text>(?!.*img).*?)</a>", RegexOptions.IgnoreCase);
                foreach (Match match in links)
                {
                    var city = new City
                    {
                        CityName = match.Groups["text"].Value,
                        Uri = new Uri("http://hotels.ctrip.com" + match.Groups["href"].Value
                    )
                    };
                    if (!cityList.Contains(city)) cityList.Add(city);//将数据加入到泛型列表
                    Console.WriteLine(city.CityName + "|" + city.Uri);//将城市名称及URL显示到控制台
                }
                Console.WriteLine("===============================================");
                Console.WriteLine("爬虫抓取任务完成！合计 " + links.Count + " 个城市。");
                Console.WriteLine("耗时：" + e.Milliseconds + "毫秒");
                Console.WriteLine("线程：" + e.ThreadId);
                Console.WriteLine("地址：" + e.Uri.ToString());
            };
            cityCrawler.Start(new Uri(cityUrl)).Wait();//没被封锁就别使用代理：60.221.50.118:8090
        }

	

### 技术探讨/联系方式

- QQ号: 276679490

- 爬虫架构讨论群：180085853


