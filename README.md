# Jd_Seckill
请安装python 3.8 运行此项目，就不会出现各种问题了

下一步打算出windows，mac系统上的安装程序

windows目前已经打包完毕，请下载`jd_maotai_20210102.zip`文件，解压，双击`main.exe`即可运行，但是仍然需要在解压文件中填写config.ini配置信息 `eid`和`fp` 还是需要填写的哦，千万不要忘记哦！！！


## 简介
通过我这段时间的使用（2020-12-12至2020-12-17），证实这个脚本确实能抢到茅台。我自己三个账号抢了四瓶，帮两个朋友抢了4瓶。
大家只要确认自己配置文件没有问题，Cookie没有失效，坚持下去总能成功的。

根据这段时间大家的反馈，除了茅台，其它不需要加购物车的商品也不能抢。具体原因还没有进行排查，应该是京东非茅台商品抢购流程发生了变化。  
为了避免耽误大家的时间，先不要抢购非茅台商品。  
等这个问题处理好了，会上线新版本。


## 暗中观察

根据12月14日以来抢茅台的日志分析，大胆推断再接再厉返回Json消息中`resultCode`与小白信用的关系。  
这里主要分析出现频率最高的`90016`和`90008`。  

### 样例JSON
```json
{'errorMessage': '很遗憾没有抢到，再接再厉哦。', 'orderId': 0, 'resultCode': 90016, 'skuId': 0, 'success': False}
{'errorMessage': '很遗憾没有抢到，再接再厉哦。', 'orderId': 0, 'resultCode': 90008, 'skuId': 0, 'success': False}
```

### 数据统计

| 案例 | 小白信用 | 90016 | 90008 | 抢到耗时 |
| ---- | ---- | ---- | ---- | ---- |
| 张三 | 63.8 | 59.63% | 40.37% | 暂未抢到 |
| 李四 | 92.9 | 72.05% | 27.94% | 4天 |
| 王五 | 99.6 | 75.70% | 24.29% | 暂未抢到 |
| 赵六 | 103.4 | 91.02% | 8.9% | 2天 |

### 猜测
推测返回90008是京东的风控机制，代表这次请求直接失败，不参与抢购。  
小白信用越低越容易触发京东的风控。  

从数据来看小白信用与风控的关系大概每十分为一个等级，所以赵六基本上没有被拦截，李四和王五的拦截几率相近，张三的拦截几率最高。  

风控放行后才会进行抢购，这时候用的应该是水库计数模型，假设无法一次性拿到所有数据的情况下来尽量的做到抢购成功用户的均匀分布，这样就和概率相关了。  

> 综上，张三想成功有点困难，小白信用是100+的用户成功几率最大。

## 主要功能

- 登陆京东商城（[www.jd.com](http://www.jd.com/)）
  - 用京东APP扫码给出的二维码
- 预约茅台
  - 定时自动预约
- 秒杀预约后等待抢购
  - 定时开始自动抢购

## 运行环境
请安装python 3.8 运行此项目
- [Python 3.8](https://www.python.org/)

## 第三方库

- 需要使用到的库已经放在requirements.txt，使用pip安装的可以使用指令  
`pip install -r requirements.txt`
- 如果国内安装第三方库比较慢，可以使用以下指令进行清华源加速
`pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/`

## 使用教程  
#### 1. 推荐Chrome浏览器
#### 2. 网页扫码登录，或者账号密码登录
#### 3. 填写config.ini配置信息 
(1)`eid`和`fp`找个普通商品随便下单,然后抓包就能看到,这两个值可以填固定的 
> 随便找一个商品下单，然后进入结算页面，打开浏览器的调试窗口，切换到控制台Tab页，在控制台中输入变量`_JdTdudfp`，即可从输出的Json中获取`eid`和`fp`。  
> 不会的话参考原作者的issue https://github.com/zhou-xiaojun/jd_mask/issues/22

(2)`sku_id`,`DEFAULT_USER_AGENT` 
> `sku_id`已经按照茅台的填好。
> `cookies_string` 现在已经不需要填写了
> `DEFAULT_USER_AGENT` 可以用默认的。谷歌浏览器也可以浏览器地址栏中输入about:version 查看`USER_AGENT`替换

(3)配置一下时间
> 现在不强制要求同步最新时间了，程序会自动同步京东时间
>> 但要是电脑时间快慢了好几个小时，最好还是同步一下吧

以上都是必须的.
> tips：
> 在程序开始运行后，会检测本地时间与京东服务器时间，输出的差值为本地时间-京东服务器时间，即-50为本地时间比京东服务器时间慢50ms。
> 本代码的执行的抢购时间以本地电脑/服务器时间为准

(4)修改抢购瓶数
> 代码中默认抢购瓶数为2，且无法在配置文件中修改
> 如果一个月内抢购过一瓶，最好修改抢购瓶数为1 
> 具体修改为：在`jd_spider_requests.py`文件中搜索`self.seckill_num = 2`，将`2`改为`1`

#### 4.运行main.py 
根据提示选择相应功能即可

#### 5.抢购结果确认 
抢购是否成功通常在程序开始的一分钟内可见分晓！  
搜索日志，出现“抢购成功，订单号xxxxx"，代表成功抢到了，务必半小时内支付订单！程序暂时不支持自动停止，需要手动STOP！  
若两分钟还未抢购成功，基本上就是没抢到！程序暂时不支持自动停止，需要手动STOP！  

## 打赏
不用再打赏了，抢到茅台的同学请保持这份喜悦，没抢到的继续加油 :)  

## 感谢
##### 非常感谢原作者 https://github.com/zhou-xiaojun/jd_mask 提供的代码
##### 也非常感谢 https://github.com/wlwwu/jd_maotai 进行的优化
