---
title: faker - 测试数据生成
tags:
  - python
  - 数据
  - 测试
categories:
  - python
date: 2020-09-05 10:50:09
---

系统测试时, 需要构造大量的测试数据, 同时这些数据最好有一定的业务含义, 比如: 姓名字段, 可以随便写个字符串"卡卡卡", 虽然能正常测试, 但体验很不好, 而类似"蔡淑英"这种就很符合业务要求了. 

[faker](https://github.com/joke2k/faker)是一个python库, 用来生成各种类型的数据, 比如姓名/地址/邮箱/电话等等, 而且支持不同语种, 比如: 姓名, 既可以生成英文的姓名"Lucy Cechtelar", 也可生成中文姓名"贺建军", 它一共支持日文/韩文/法文等30多种语言. 



## Java版API

除了python版, 类似的还有Java/PHP/Ruby等, 先简单说下[Java](https://github.com/DiUS/java-faker)版.

先引入maven依赖

```xml
<dependency>
    <groupId>com.github.javafaker</groupId>
    <artifactId>javafaker</artifactId>
    <version>1.0.2</version>
</dependency>
```

api使用

```java
Faker faker = new Faker();
faker.name().fullName();    // Gordon Lemke
faker.address().streetAddress();    // 3980 Isaiah Square

// 生成中文类数据
faker = new Faker(new Locale("zh-CN"));
faker.name().fullName();    // 胡靖琪
faker.address().streetAddress();    // 赖巷43213号
```



## python版API

下面是python版的API列表

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from random import randint
from collections import OrderedDict
from datetime import datetime, date
from faker import Faker
from faker.providers import BaseProvider, internet     

# -----------  faker.providers.BaseProvider -----------
fake = Faker();
# 格式化输出 数字+字符 0-9
fake.bothify(letters='ABCDE'    #93 AA
fake.bothify(text='Product Number: ????-########')    #aGEt-22666920
fake.bothify(text='Product Number: ????-########', letters='ABCDE')    #VDECE-73691847
# 格式化输出 字符
fake.lexify(text='Random Identifier: ??????????')  # Random Identifier: HsoUenwndr
fake.lexify(text='Random Identifier: ??????????', letters='ABCDE')   # Random Identifier: AECDBBECEB
# 格式化输出 数字     #0-9 %1-9 !0-9空格 @1-9空格
fake.numerify(text='Intel Core i%-%%##K vs AMD Ryzen % %%##X')   # Intel Core i4-6966K vs AMD Ryzen 7 1612X
# 格式化输出16进制数
fake.hexify(text='MAC Address: ^^:^^:^^:^^:^^:^^')   # MAC Address: a9:c4:39:79:ba:4c
fake.hexify(text='MAC Address: ^^:^^:^^:^^:^^:^^', upper=True)   # MAC Address: DC:CC:19:2C:4F:A1

# 组成随机列表
fake.random_choices(elements=('a', 'b', 'c', 'd'))  # ['a', 'd', 'b', 'b']
fake.random_choices(elements=('a', 'b', 'c', 'd'), length=3) # ['c', 'd', 'd']
# 出现概率
fake.random_choices(elements=OrderedDict([("a", 0.45), ("b", 0.35), ("c", 0.15), ("d", 0.05), ]))    
# 随机列表, 不重复
fake.random_sample(elements=('a', 'b', 'c', 'd', 'e', 'f'), length=3)
# 随机元素
fake.random_element(elements=('a', 'b', 'c', 'd'))   # d
fake.random_element(elements=OrderedDict([("a", 0.45), ("b", 0.35), ("c", 0.15), ("d", 0.05), ]))    # 有概率
# 随机元素不重复
fake.random_elements(elements=('a', 'b', 'c', 'd'), unique=True) #['b', 'a', 'c']
fake.random_elements(elements=OrderedDict([("a", 0.45), ("b", 0.35), ("c", 0.15), ("d", 0.05), ]), length=2, unique=True)

# 随机数字
fake.random_int()    # 1127
fake.random_int(min=100, max=200) # 191
fake.random_int(min=0, max=15, step=3) # 6
# 指定长度的数字
fake.random_number()    # 长度随机 435701
fake.random_number(digits=3)    # 长度固定  785
# 指定范围数字
fake.randomize_nb_elements(number=100)   # 默认 60% - 140% 139
fake.randomize_nb_elements(number=100, le=True)  # 60% - 100% 
fake.randomize_nb_elements(number=100, ge=True)  # 100% -140%
fake.randomize_nb_elements(number=100, le=True, max=80)  # 60% - max
fake.randomize_nb_elements(number=100, ge=True, min=120) # min - 140%

# 1位随机数  0-9
fake.random_digit() # 3
# 1位随机数  1-9
fake.random_digit_not_null() # 4
# 1位随机数或空格  1-9
fake.random_digit_not_null_or_empty() # 8
# 1位随机数或空格  0-9
fake.random_digit_or_empty() # 9

# 随机字母 a-z A-Z
fake.random_letter()  # a
# 随机小写字母
fake.random_lowercase_letter()   # i
# 随机大写字母
fake.random_uppercase_letter()   # D
# 随机字母列表
fake.random_letters()   # ['G', 'B', 'O', 'p', 'V', 'Q', 'F', 'P', 'n', 'H', 'c', 'N', 'r', 'r', 'h', 'A']
fake.random_letters(length=6)    # ['A', 'E', 'W', 'Q', 'r', 'A']

# 语言代码
fake.language_code()  # uk
# 地区代码
fake.locale()    # zh_HK

# fake.add_provider(internet)    
fake.ipv4_private()

# -----------  faker.providers.address.Provider -----------
fake = Faker("zh_CN"
# 随机地址
fake.address()   # 四川省柳州市沈河林街w座 521082
# 随机楼号
fake.building_number()
# 随机城市
fake.city()  # 宁德市
# 随机行政单位  市县
fake.city_suffix()   # 市
# 随机国家
fake.country()   # 塔吉克斯坦
# 随机国家代码
fake.country_code() # MG
# 随机邮编
fake.postcode()  # 406008
# 随机街道地址
fake.street_address()    # 常街O座
# 随机路名
fake.street_name()   # 宜都路
# 街道后缀
fake.street_suffix() # 路

# -----------  faker.providers.color.Provider -----------
# HSV
# 随机某个色系颜色  monochrome灰度  red orange yellow green blue purple pink
fake.color(hue='monochrome')    # #a80a14
# 随机亮色
fake.color(luminosity='light')   # #86b3ef
# 颜色格式 hex rgb hsv hsl
fake.color(hue=(100, 200), color_format='rgb')   # rgb(176, 227, 244)
fake.color(hue=135, luminosity='dark', color_format='hsv')   # hsv(135, 95, 46)
fake.color(hue=(300, 20), luminosity='random', color_format='hsl')   # hsl(195, 16, 2
# 随机颜色名称
fake.color_name()    # MediumOrchid
# 随机16进制颜色
fake.hex_color()     # #a65ad6
# 随机rgb颜色
fake.rgb_color()     # 27,14,255
# 随机css格式rgb颜色
fake.rgb_css_color()     # rgb(59,208,119
# 随机安全色名称
fake.safe_color_name()   # fuchsia
fake.safe_hex_color()    # #668800

# -----------  faker.providers.company.Provider -----------
# 随机公司名称
fake.company()   # 华远软件科技有限公司
# 随机信用卡过期时间
fake.credit_card_expire()    # 03/21
# 完整信用卡
fake.credit_card_full()      # American Express\n博 王\n344778599396849 07/28\nCID: 3995
# 信用卡号
fake.credit_card_number()    # 6011562560356696
# 随机 am pm
fake.am_pm()     # AM
# 随机日期  19700101 - 今天
fake.date()      # 2010-12-19
# 随机日期 30年前 - 今天
fake.date_between()  # 1993-05-06
# 随机日期 1年前 - 今天
fake.date_between(start_date='-1y', end_date='today')    # 2020-02-11
# 两个时间中的某个时间  Date / Datetime
fake.date_between_dates(date_start=date(2018, 1, 1))  # 2020-05-03
fake.date_between_dates(date(2010, 1, 1), date(2015, 1, 1))  # 2013-01-06
# 某个随机年龄对应的日期
fake.date_of_birth(minimum_age=18, maximum_age=30)   # 2001-10-23
# 这个月的某个日期
fake.date_this_month()
fake.date_this_month(before_today=True, after_today=True)
# 今年的某个日期
fake.date_this_year()
fake.date_this_year(before_today=True, after_today=False)
# 时间 10101 - 现在
fake.date_time_between() # 2009-11-07 04:47:17
fake.date_time_between(start_date='-1y', end_date='now') # 2020-08-31 05:38:55
# 某个范围内的时间
fake.date_time_between_dates()
fake.date_time_between_dates(datetime(2013, 2, 8, 15, 53, 19), datetime(2016, 6, 1, 23, 33, 57))
# 一个月中的某天
fake.day_of_month()  # 18
# 某个月
fake.month()  # 09
# 某年
fake.year()  # 2006
# 某个月份名
fake.month()  # August
# 周几
fake.day_of_week()   # Tuesday
# 未来某天
fake.future_date()       # 2020-09-25
fake.future_date(end_date='+3d')     # 2020-09-05
fake.future_datetime(end_date='+3d')     # 2020-09-04 15:45:59
# 过去某天
fake.past_date()     
fake.past_date(start_date='-3d')
fake.past_datetime(start_date='-3d')
# 随机时区
fake.timezone()  # Europe/Bucharest
# 随机时间戳
fake.unix_time()
fake.unix_time(start_datetime=datetime(2020,1,1))

# -----------  faker.providers.file.Provider -----------
# 文件后缀
fake.file_extension()       # pptx
# 文件名
fake.file_name()     # 价格.mp3
# 指定文件类型   category: audio|image|office|text|video
fake.file_name(category='image', extension='jpeg')   # 起来.jpeg
# 文件路径
fake.file_path()     # /增加/那个.html
fake.file_path(depth=3, category='image', extension='jpeg')  # /什么/人员/详细/欢迎.jpeg
# 网络传输文件类型
fake.mime_type()     # application/xhtml+xml
# category – application|audio|image|message|model|multipart|text|video
fake.mime_type(category='image')     # image/png

# -----------  faker.providers.company.Provider -----------
fake.coordinate()    # 113.507229
# 纬度
fake.latitude()     # -76.139686
# 经度
fake.longitude()     # 7.562154
# 纬度 + 经度
fake.latlng()    # (Decimal('14.9397975'), Decimal('-38.213072')
# 城市坐标
fake.local_latlng()  #('39.33427', '-76.43941', 'Middle River', 'US', 'America/New_York')
fake.local_latlng(country_code='CN') # ('30.24706', '115.04814', 'Huangshi', 'CN', 'Asia/Shanghai'
# 陆地坐标
fake.location_on_land()  # ('41.6764', '-91.58045', 'Coralville', 'US', 'America/Chicago'

# -----------  faker.providers.internet.Provider -----------
# 公司邮箱
fake.company_email()     # luojie@junyong.cn
fake.ascii_company_email()   # ihou@gangwang.cn
# 邮箱
fake.email()     # rxiao@hotmail.com
fake.free_email()     # xiashao@gmail.com
fake.ascii_email()   # ysu@hotmail.com
fake.safe_email()     # vchang@example.org
fake.ascii_free_email()  # jing12@yahoo.com
fake.ascii_safe_email()  # chao59@example.net

# 域名
fake.domain_name()   # xiuyingping.cn
fake.domain_name(levels=2)   # junjing.61.net
fake.safe_domain_name()   # example.org
fake.free_email_domain()   # yahoo.com
fake.hostname()   # laptop-53.min.cn
fake.hostname(levels=2)   # srv-04.22.sx.cn
# 顶级域名
fake.tld()   # com

# http方法
fake.http_method()   # DELETE

# 图片链接
fake.image_url()     # https://dummyimage.com/865x559
fake.image_url(width=640, height=480)    # https://www.lorempixel.com/640/480

# ip地址
fake.ipv4()  # 111.229.96.116
fake.ipv4(network=False, address_class='a', private=True)    # 10.158.89.21
fake.ipv4_private()  # 192.168.181.245
fake.ipv4_public()   # 159.233.210.51
fake.ipv6()  # cf97:ce3f:1575:abf3:735:84d0:2d59:db22
# mac地址
fake.mac_address()   # c6:d2:bb:f8:41:99
# 端口号
fake.port_number()   # 3588
fake.port_number(is_system=False, is_user=True, is_dynamic=False)   # 3588
# url地址
fake.url()   # http://yang.cn/
fake.uri()   # https://www.leitao.cn/tags/tag/app/register/
# url路径
fake.uri_path(deep=3)    # posts/categories/categories

# 用户名
fake.user_name() # xiuying06
# 密码
fake.password()  # 630Td#lM%P
fake.password(length=10, special_chars=False, digits=True, upper_case=True, lower_case=True)  # H0ZAAvnq02

# -----------  faker.providers.lorem.Provider -----------
# 一段文本
fake.paragraph() # 相关设计工具提高你们.结果建设两个浏览.
fake.paragraph(nb_sentences=10, variable_nb_sentences=True, ext_word_list=None)
# 多段文本列表
fake.paragraphs()
fake.paragraphs(nb=10, ext_word_list=None)
# 一句话
fake.sentence()
fake.sentence(nb_words=10, variable_nb_words=True, ext_word_list=None)
# 几句话列表
fake.sentences()
fake.sentences(nb=10, ext_word_list=None)
# 文本
fake.text()
fake.text(max_nb_chars=100, ext_word_list=None))
fake.texts()
fake.texts(nb_texts=10, max_nb_chars=200, ext_word_list=None)
# 单词
fake.word()
fake.words(nb=10, ext_word_list=None, unique=False)

# -----------  faker.providers.misc.Provider -----------
# boolean 
fake.boolean()   # True
fake.boolean(chance_of_getting_true=25)  # False

# json [(字段名, 方法名, {方法参数})]
fake.json(data_columns=[('id', 'pyint', {'max_value':20})], num_rows=3)  # [{"id": 3}, {"id": 19}, {"id": 9}]
# {"id": 156, "details": {"name": "min20@yangyuan.cn"}}
fake.json(data_columns=[('id', 'pyint'), ('details', (('name', 'email'), ))], num_rows=1)
# {"id": 6692, "details": ["\u9a6c\u5e06", "\u5510\u6d9b"]}
fake.json(data_columns=[('id', 'pyint'), ('details', [(None, 'user_name'), (None, 'user_name')])], num_rows=1)


# md5
fake.md5()   # 2f265a4ae94544ef1412c9983c0af282
fake.md5(raw_output=True)   # 二进制  b'\n\x8e\x06\xa2\xe0\xbczS,\xd4j\xb9k0\r.'
# sha1
fake.sha1()  # 544667252ceb900d85034313571c45ec4ee50dca
fake.sha256()  # 49c96b62cc664fc26eec3e7e412c0a214130acfa41d37b57bfe7eccf5c519581
# uuid
fake.uuid4()     # b0b03cfb-420d-4bc8-a74f-0e2849b19e97

# 压缩文件
fake.tar(uncompressed_size=256, num_files=4, min_file_size=32))
fake.zip(uncompressed_size=256, num_files=4, min_file_size=32)
# 指定压缩格式   deflate|gzip|gz  bzip2|bz2  lzma|xz
fake.zip(uncompressed_size=256, num_files=32, min_file_size=4, compression='bz2') 

# -----------  faker.providers.person.Provider -----------
# 名字
fake.name()  # 蔡淑英
# 女性名字
fake.name_female()  # 刘倩
# 男性名字
fake.name_male()  # 贺建军
# 姓
fake.last_name()     # 朱
# 名
fake.first_name()    # 斌
# 女性名字
fake.first_name_female()     # 秀兰
# 男性名字
fake.first_name_male()     # 文

# -----------  faker.providers.phone_number.Provider -----------
# 国家号码前缀
fake.country_calling_code()  # +86
# 电话号码
fake.phone_number()  # 18578143124

# -----------  faker.providers.profile.Provider -----------
# 个人信息
# {'job': '市场/营销', 'company': '昂歌信息信息有限公司', 'ssn': '63220019390724127X', 
# 'residence': '重庆市超市丰都翟街p座 693155', 'current_location': (Decimal('33.961661'),
# Decimal('-177.155429')), 'blood_group': 'B-', 'website': ['http://www.chaogao.cn/'],
# 'username': 'guiyingyao', 'name': '岳兰英', 'sex': 'F', 'address': '吉林省成都市沙湾何路m座 517600', 
# 'mail': 'limo@yahoo.com', 'birthdate': datetime.date(2013, 4, 10)}
fake.profile()
# {'username': 'nliu', 'name': '李燕', 'sex': 'F', 'address': '广西壮族自治区慧县萧山罗路m座 976267', 
# 'mail': 'gang71@gmail.com', 'birthdate': datetime.date(2009, 5, 5)}
fake.simple_profile()

# -----------  faker.providers.python.Provider -----------
fake.pybool()        # False
fake.pydecimal()     # Decimal('97.4104')
fake.pydict()
fake.pyfloat()   # 97.4104
fake.pyfloat(left_digits=None, right_digits=None, positive=False, min_value=None, max_value=None)
fake.pyint()     # 6890
fake.pyint(min_value=0, max_value=9999, step=1)
fake.pyiterable()
fake.pylist()
fake.pyset()
fake.pystr()
fake.pystr_format()
fake.pystruct()
fake.pytuple()


# 多语种
fake = Faker(['zh_CN', 'en_US'])
for _ in range(5):
    fake.name()

# 自定义数据
fake = Faker()
class MyProvider(BaseProvider):
    def province(self):
        provinces = ["北京市", "天津市", "河北省", "内蒙古", "山西省", "辽宁省",
         "吉林省", "黑龙江省", "上海市", "江苏省", "山东省", "安徽省", "福建省", 
         "浙江省", "江西省", "广东省", "广西", "海南省", "重庆市", "四川省", 
         "云南省", "贵州省", "西藏", "河南省", "湖北省", "湖南省", "陕西省", 
         "青海省", "宁夏", "新疆", "甘肃省", "香港", "澳门", "台湾"]
        return provinces[randint(0, len(provinces) - 1)]
fake.add_provider(MyProvider)
fake.province()
```



