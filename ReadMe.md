		这是一款Golang编写的漏洞扫描工具，非开发人员也可以根据yaml模板比较轻易的学会编写yaml格式的poc，不至于一款工具太依赖于作者的poc更新，可以自己将平时收集到的poc添加到工具中使用。（golang初学者编写，勿喷）

### 使用说明

​		使用方法很简单，下载releases中的压缩包，运行main.exe即可启动GUI界面，在扫描模块配置扫描的目标，超时，现成，代理参数，在选项模块勾选需要扫描的漏洞。

### 自定义poc

​		通过向poc目录对应的子目录下添加yaml文件，GUI工具页面会自动获取poc目录的结构生成对应的漏洞选项(yaml文件名会直接用于GUI页面中的选项名)。

### demo文件介绍

​		demo1.yaml --- 普通常规的通过多个请求和对应的返回包进行校验，如果所有请求包返回结果满足设定的条件则判断存在该漏洞

​		demo_dnslog.yaml  --- 用于需要dnslog来结合判断漏洞是否存在的demo，主要原理是在demo1的基础上第一个请求为获取dnslog临时域名和cookie，最后一个请求则获取dnslog平台对应的临时域名有没有dns请求记录。

​		demo_fileupload.yaml  --- 用于需要读取某一文件内容进行上传（例如zip、c等文件)



### yaml文件参数介绍：

```
    # 可替换参数说明
    #   {{payload}} ------- 用于跟rules-requests-payload中的数据进行替换
    #                     地址处理
    #    eg: url = http://192.168.12.23:8801/ddxxmm/index.html
    #   {{rootUrl}}为根目录（末尾无斜线） url >> http://192.168.12.23:8801
    #	{{url}}为输入的url本身不做处理
    #   {{host}}为url中的host部分    url >>  192.168.12.23:8801
    #   {{hostName}}为url中的host去掉端口部分    url >>  192.168.12.23
    #   {{port}}为url中的端口部分   url >>  8801
    #
    #                     时间处理
    #   {{year}}  为当前年份去掉前两位，如当前为2022年，{{year}}的值为22
    #   {{month}} 为当前月份，如当前为2022年3月，{{month}}的值为03
    #   {{day}}   为当前月份，如当前为2022年3月9日，{{day}}的值为09
    #
    #                     数据处理
    #   {{base64(test)}} 的值为将test经过base64编码后的值，test为可变字符串
    #   {{md5(test)}} 的值为将test经过md5 32位加密后的值，test为可变字符串
    #   {{random(6)}} 随机生成6位的数字字母大小写组合

info:
  # 跟yaml文件名相同
  vulId: tianqing_fileupload
  # 漏洞描述
  detail: phpinfo敏感信息泄露漏洞
  # fofa语法
  fofaQuery: app="Thinkphp"
# 一个数组，用来存放从请求返回包中正则匹配到的数据，在后面请求需要用到时使用{{variable[x]}}来指定
variable:
  -
rules:
  # 漏洞验证请求,如果需要多个请求进行组合验证则可以写多条
  - request:
      # 一个数组payload用于替换request中的{{payload}}
      payload:
        - "ipconfig"
        - "ifconfig"
      path: "{{rootUrl}}/phpinfo.php"
      # 请求方法，暂时可以有POST、GET、PUT、DELETE
      method: POST
      # 请求头，字典格式
      headers:
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.67 Safari/537.36
        Referer: "{{rootUrl}}"
        Content-type: multipart/form-data; boundary=----WebKitFormBoundaryLx7ATxHThfk91oxQ
        Accept-Encoding: gzip
      # 是否允许跳转
      redirect: false
      # 请求包格式，text对应nic的RAW，json对应data(暂时仅支持text)
      dataTtpe: text
      # 请求包主体内容(data和files仅存在一个)
      data: "{{payload}}"
      # 如果是上传文件请求
      files:
        # 表单中的的name参数
        name: file
        # 上传的文件地址
        filePath: shell.php
        # 表单的fileName参数
        fileName: test.png
        # 表单的Content-Type参数
        contentType: application/octet-stream
    # 检查是否成功，checks里面如果有多个type则这里指定同时满足还是满意一个即可
    checksCondition: and
    checks:
      # 检查类型：string(字符串校验)，status_code(状态码校验)，regex(正则校验)
      - checkType: string
        # 校验内容
        desireds:
          - PHP Extension
          - PHP Version
          - phpinfo
        # 校验位置：body，header
        place: body
        # 校验内容如果存在多个则指定同时满足还是满意一个即可
        condition: and

      - checkType: status
        # 校验内容
        desireds:
          - 200
        # 校验内容如果存在多个则指定同时满足还是满意一个即可
        condition: and
```





较多poc暂未加入（太费时间了），漏洞利用模块暂未开发，敬请期待！！！！
