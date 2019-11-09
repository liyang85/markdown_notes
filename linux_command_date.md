# Linux command: date

## 在不同时区之间转换时间

### 方法一（不推荐）

```bash
# 需要转换的时间

$ TZ=EST5EDT date -d '2019/6/1 20:00'
Sat Jun  1 20:00:00 EDT 2019

# 第一步：把特定时区的时间转换为与时区无关的 Unix 时间戳

$ TZ=EST5EDT date -d '2019/6/1 20:00' +%s
1559433600

# 第二步：把 Unix 时间戳转换为目标时区的时间

$ TZ=Asia/Shanghai date -d @1559433600
Sun Jun  2 08:00:00 CST 2019
```

这里的 `EST5EDT` 可以说是 ET 时区的一种。关于 ET 时区，有以下问题需要注意：

> The term **Eastern Time (ET)** is often used to denote the local time in areas observing either Eastern Daylight Time (EDT) or Eastern Standard Time (EST).
>
> - Eastern Standard Time is 5 hours behind UTC (Coordinated Universal Time).
> - Eastern Daylight Time is 4 hours behind UTC.
>
> From: <https://www.timeanddate.com/time/zones/et>

由此可见，ET 时区只是一个笼统的说法，使用时需要明确到底是 EST 还是 EDT（具体差异在后文详述），因此上面的命令中才会出现 `TZ=EST5EDT` 这个特殊的时区名称。

### 方法二（推荐）

```bash
# https://superuser.com/a/164354/579057

# 台北时间是 2019/10/1 22:00，纽约时间是什么时候？
# 此处必须使用 双引号 包围时区名称，否则会报错！

$ TZ=America/New_York date --date='TZ="Asia/Taipei" 2019/10/1 22:00'
Tue Oct  1 10:00:00 EDT 2019
```

更简短的语法：

```bash
# UTC 时间是 2019/10/1 22:00，纽约时间是什么时候？

$ TZ=America/New_York date -d '2019/10/1 22:00 UTC'
Tue Oct  1 18:00:00 EDT 2019

# 时区写成 UTC 或 UTC+0 都可以

$ TZ=America/New_York date -d '2019/10/1 22:00 UTC+0'
Tue Oct  1 18:00:00 EDT 2019

# New York 属于 UTC-4 时区

$ TZ=America/New_York date +%z
-0400
```

## 使用没有歧义的时区名称

某些时区缩写很容易产生歧义，例如 CST 就可以表示多个时区：

- China Standard Time (UTC+8)
- Cuba Standard Time (UTC-5)
- Central Standard Time (UTC-6)

因此，编写 Shell 脚本的时候，我们应该**使用标准的、没有歧义的时区名称**，也就是「时区信息数据库（tz database）」定义的时区名称，例如 `America/New_York` , `Europe/Paris` , `Asia/Shanghai` 。

> The **tz database** is a collaborative compilation of information about the world's time zones, **primarily intended for use with computer programs and operating systems**. Paul Eggert is its current editor and maintainer, with the organizational backing of ICANN. The tz database is also known as **tzdata**, the **zoneinfo database** or **IANA time zone database**, and occasionally as the **Olson database**, referring to the founding contributor, Arthur David Olson.
>
> Its uniform naming convention for time zones, such as America/New_York and Europe/Paris, was designed by Paul Eggert. The database attempts to record historical time zones and all civil changes since 1970, the Unix time epoch. It also includes transitions such as daylight saving time, and also records leap seconds.
>
> From: <https://en.wikipedia.org/wiki/Tz_database>

Linux 的 tzdata 存放在 `/usr/share/zoneinfo/` 目录：

```bash
$ ls /usr/share/zoneinfo/
+VERSION     CST6CDT  Europe/    Hongkong   MST       Portugal   WET
Africa/      Canada/  Factory    Iceland    MST7MDT   ROC        Zulu
America/     Chile/   GB         Indian/    Mexico/   ROK        iso3166.tab
Antarctica/  Cuba     GB-Eire    Iran       NZ        Singapore  posixrules
Arctic/      EET      GMT        Israel     NZ-CHAT   Turkey     zone.tab
Asia/        EST      GMT+0      Jamaica    Navajo    UCT
Atlantic/    EST5EDT  GMT-0      Japan      PRC       US/
Australia/   Egypt    GMT0       Kwajalein  PST8PDT   UTC
Brazil/      Eire     Greenwich  Libya      Pacific/  Universal
CET          Etc/     HST        MET        Poland    W-SU
```

也可以通过 `systemd` 提供的 `timedatectl list-timezones` 命令列出所有时区：

```bash
$ timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
Africa/Bamako
...
UTC
```

通过 `timedatectl` 命令列出时区的时候，除了 `UTC` ，其它时区名称都是「大洲/城市」这种统一的风格。

在所有这些时区中，许多已经被弃用（deprecated），只是为了兼容性而仍然保留，例如 `EST` 和 `EST5EDT` ，新编写的脚本不应该使用这些名称。

> 详细的时区名称状态信息参见：
> <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>

也可以从 [TimeAndDate.com](https://www.timeanddate.com/) 非常直观的查看[全球时区地图](https://www.timeanddate.com/time/map/)。

## 夏时制

夏时制指的是：在天亮较早的夏季，**人为的将时间调快一小时**，使人们早起早睡，减少照明量，**充分利用日光，从而节约照明用电**（实际效果备受争议），因此也叫做「日光节约时间（Daylight Saving Time, DST）」。它还有以下叫法：

- 夏令时
- Daylight time (United States)
- Summer time (United Kingdom, European Union, and others)

夏时制与全世界普遍使用的「时区」概念不一样，**它是一种行政管理手段**，仅仅在部分国家使用：

- 邻近赤道的地区通常不使用夏时制。
- 亚洲、非洲一般不使用夏时制。
- 中国曾经短暂使用夏时制，从 1992 年暂停使用。
- [澳大利亚的部分州使用夏时制](https://en.wikipedia.org/wiki/Daylight_saving_time_in_Australia)。
- 美国只有极少数的州**不使用**夏时制。
  - Arizona (except the Navajo Nation) and Hawaii do not use DST.

即使在那些使用夏时制的国家，每年的启用日期和终止日期也不同：

| 国家     | 洲别   | 半球   | 启用夏令时间                       | 终止夏令时间                        |
| -------- | ------ | ------ | ---------------------------------- | ----------------------------------- |
| 澳大利亚 | 大洋洲 | 南半球 | 10 月第一个星期日的 2 时           | 4 月第一个星期日的 2 时             |
| 美国     | 北美洲 | 北半球 | 3 月第二个星期日                   | 11 月第一个星期日                   |
| 英国     | 欧洲   | 北半球 | 3 月最后一个星期日的 UTC 时间 1 时 | 10 月最后一个星期日的 UTC 时间 1 时 |

> - 数据来自 <https://en.wikipedia.org/wiki/Daylight_saving_time_by_country> 。
> - 表格由 <http://www.tablesgenerator.com/markdown_tables> 生成。

了解以上内容之后，我们就可以进一步对比标准时间和夏时制的区别了。以 EST 和 EDT 为例：

```bash
# CST = China Standard Time (UTC+8)

$ date --date='TZ="EST" 2019/10/1 18:00'
Wed Oct  2 07:00:00 CST 2019

$ date --date='TZ="EST5EDT" 2019/10/1 18:00'
Wed Oct  2 06:00:00 CST 2019
```

### 中国暂停使用夏时制的原因

> 中国虽然横跨五个时区，但在全国范围内统一使用东八区时间（UTC+8），俗称「北京时间」，也就是东经 120 度线所在的时间。从地图上看，**东经 120 度线离上海最近**，在北京东边，沈阳西边。
>
> - 对于江浙沪地区，这个时区是最合适的。
> - 对于东北地区来说，晚了半个小时左右。
> - 对于华北、中原、湘赣、珠三角地区来说，这一时区早了半个小时。
> - 对于陕西、四川、云南来说，这一时区早了一个小时。
>
> 因此，**对于中国的大部分人口密集地区，包括华北、中原、湘赣、珠三角、陕西、四川、云南来说，东八区时间相对于本地来说是提前了半小时到一小时。对于这些地区来说，一直实施的都是夏令时**，再实施夏令时就相当于提前两个小时，效果不好。
>
> From: <https://www.zhihu.com/question/20309772/answer/93028404>
