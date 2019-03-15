# wxapp-boot-time

微信小程序启动时间的测量方案，借助Appium测试框架、ffmpeg视频处理、pyssim图片相似度判断等开源工具，整体误差在 20~50ms左右。

## 一、背景
小程序的启动耗时一直是我们非常关注的指标，目前这个指标的主要衡量标准是微信官方后台的报表，这个数据具有较高的参考意义，但是也有几个问题：

- 只有正式版的数据
- 指标的定义比较模糊，会出现 下载耗时 + 渲染耗时 ！= 总启动耗时的情况
- 不够细化，只有平台、网络类型区分

简而言之它只是一个总体的参考，并不能提供很细化的优化建议。我们需要一个在**测试阶段**就能确定、**恒定测试标准**下的“准确”启动耗时，这样我们在发布前就知道启动时间是变好了还是差了，甚至能知道和竞品的耗时差距。

于是本方案应用而生，其实也是业内通用的方案，基于录屏+图片分析，开源出来希望能够帮助更多人。

## 二、技术方案
### 2.1 网络限速
小程序启动过程会有一个网络下载过程，需要尽量排除网络的影响。这是保证测试数据稳定的大前提。

一种思路是在手机端限制网速，但是需要ROOT，操作麻烦很多。目前采用了更简单的方案：

- 路由器端限制设备网速 100KB/s（找运维配置）
- 凌晨5~6点网络低谷时执行测试
- 连续测试5次，去除最大值最小值后取平均值

### 2.2 计算方法
在启动小程序（通过下拉菜单启动）时同步录屏，将视频逐帧分解为图片，然后再通过对比图片分析计算出启动时间。

详细处理流程参考代码实现，可视具体小程序调整相关的相似度因子。

### 2.3 持续集成
将脚本放到 jenkins 进行持续集成，数据汇总到后台生成报表（此部分未开源）

## 三、使用教程
### 3.1 安装[Appium](http://appium.io/)并启动服务
用于驱动手机自动化操作，建议在服务器端运行此服务，运行服务在Mac、Windows上测试通过

需要安装Android SDK、Java等环境，推荐安装最新稳定版

### 3.2 安装 [ffmpeg](https://www.ffmpeg.org/download.html) 命令

Mac可通过 brew 安装，Windows需要下载安装包自行安装。

### 3.3 安装Python3及依赖

此脚本仅在 Python3 上测试通过，具体依赖列表参考 `requirements.txt`

推荐安装最新稳定版

### 3.4 修改配置运行服务

#### 3.4.1 修改如下配置

```python
# Appium 服务地址
EXECUTOR = 'http://127.0.0.1:4723/wd/hub'

# 被测设备信息
ANDROID_CAPS = {
    'platformVersion': '7.0',
    'deviceName': '0915f911a8d02504',
}

# 被测小程序名
WXAPP = "有车以后"

```

#### 3.4.2 放置小程序启动时间结束位置图片到对应位置

例如 `reference/有车以后/t0_end.png` ，可从录屏分解的图片中挑选一张作为结束位置

代码库里默认已经防止了 `有车以后` 和 `拼多多` 两个小程序的结束位置图片，可供参考

#### 3.4.3 启动脚本

直接运行即可 `python wxapp_boot_time.py`

```
[2019-03-15 17:51:07,920] root:INFO: 0002.png 相似度：0.999981
[2019-03-15 17:51:08,432] root:INFO: 0003.png 相似度：0.999973
[2019-03-15 17:51:08,948] root:INFO: 0004.png 相似度：0.999970
...
[2019-03-15 17:52:11,151] root:INFO: 0120.png 相似度：0.970193
[2019-03-15 17:52:11,677] root:INFO: 0121.png 相似度：0.967654
[2019-03-15 17:52:11,677] root:INFO: 开始位置：5，结束位置：121，本次启动耗时：2320毫秒
```



## 四、微信交流群
请扫码加群，如二维码失效，可加管理员 `richshaw` 申请入群，备注`小程序测试`

## 五、版权声明
[有车以后](http://youcheyihou.com/)测试组荣誉出品，如果对您项目有帮忙，欢迎Star，开源声明 [The 3-Clause BSD License](https://opensource.org/licenses/BSD-3-Clause) 

