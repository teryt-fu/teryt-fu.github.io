---
title: 给二弟三弟
date: 2019-12-30 14:44:58
tags:
- 测试
categories:
- 测试
---
1. **python安装库**
   - `pip install requests`
   - `pip install robotframework`
   - `pip install robotframework-requests`
   - `pip install robotframework-jsonschemalibrary`
   - `pip install robotframework-jsonvalidator`
2. **python利用requests库访问网址代码示例**
   ```
     # -*- coding: utf-8 -*-

     import requests

     # def getbaidu():
     #     url = "http://www.baidu.com"
     #     response = requests.get(url)
     #     print(response.status_code)
     #     # print(response.text)
     #     return response

     def hefeng():
         url = "https://free-api.heweather.net/s6/weather/now?key=yourkey&location=wuhan"
         response = requests.get(url)
         return response
   ```
3. **robot文件编写测试用例代码示例**
   ```
        ***Settings***
        Documentation    Test
        Library    RequestsLibrary
        Library    OperatingSystem
        Library    Collections
        Library    json
        Library    py_robot.py
        # Library    JSONSchemaLibrary    test_schema
        ***Test Cases***
        # test case01:测试百度返回状态码
        #     [Documentation]    test GET Baidu
        #     ${resp}=    py_robot.getbaidu  # 调用文件.函数
        #     Should Be Equal As Strings    ${resp.status_code}    200

        # test case02:测试百度返回错误状态码
        #     ${resp}=    py_robot.getbaidu
        #     Should Be Equal As Strings    ${resp.status_code}    404

        test case03:测试和风天气接口API
            [Documentation]    测试和风天气接口的返回数据
            ${resp}=    py_robot.hefeng
            Should Be Equal As Strings    ${resp.status_code}    200
            ${response}    To Json    ${resp.text}
            ${loca}    Set Variable    ${response['HeWeather6'][0]['basic']['location']}
            Should Be Equal As Strings    ${loca}    武汉

   ```
