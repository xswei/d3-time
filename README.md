# d3-time

在可视化时间序列数据、分析时间模式或处理一般时间时，常规时间单位的不规则性很快就变得明显起来。在 [Gregorian calendar(公历)](https://en.wikipedia.org/wiki/Gregorian_calendar) 中，大多数月份有 `31` 天但是有些月份只有 `28` 或者 `29`、`30` 天。大多数年份有 `365` 天但是 [leap years(闰年)](https://en.wikipedia.org/wiki/Leap_year) 有 `366` 天。在 [daylight saving(夏令时)](https://en.wikipedia.org/wiki/Daylight_saving_time) 中一天可能有 `23` `25` 小时。更复杂的是世界各地的夏时制不同。

由于这些时间特性，执行看似微不足道的任务可能会很困难。例如，如果你想计算两个日期之间相隔的天数，你可能会简单的以 `24` 小时为一天去计算：

```js
var start = new Date(2015, 02, 01), // Sun Mar 01 2015 00:00:00 GMT-0800 (PST)
    end = new Date(2015, 03, 01); // Wed Apr 01 2015 00:00:00 GMT-0700 (PDT)
(end - start) / 864e5; // 30.958333333333332, oops!
```

但是使用 [d3.timeDay](#timeDay).[count](#interval_count) 会更容易:

```js
d3.timeDay.count(start, end); // 31
```

[day](#day) [interval](#api-reference) 是 `d3-time` 提供的方法之一. 常用的时间单位比如 [hours](#timeHour), [weeks](#timeWeek), [months](#timeMonth), *etc.* 都有对应的计算边界日期的方法。例如 [d3.timeDay](#timeDay) 通常会以当地时间的午夜作为一天的开始。为了 [rounding](#interval_round) 和 [counting](#interval_count)，间隔可以用来生成一组边界日期。比如计算当前月的每一个周日:

```js
var now = new Date;
d3.timeWeek.range(d3.timeMonth.floor(now), d3.timeMonth.ceil(now));
// [Sun Jun 07 2015 00:00:00 GMT-0700 (PDT),
//  Sun Jun 14 2015 00:00:00 GMT-0700 (PDT),
//  Sun Jun 21 2015 00:00:00 GMT-0700 (PDT),
//  Sun Jun 28 2015 00:00:00 GMT-0700 (PDT)]
```

`d3-time` 模块不会实现自己的日历系统。它仅仅是基于 `ECMAScript` 的 [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) 实现了一些方便的数学计算 `API`. 因此，它忽略了闰秒，只能与当地时区和 [Coordinated Universal Time(世界协调时)](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) (UTC)一起工作。

这个模块可以被 `D3` 的时间比例尺用来生成合理的刻度，通过 `D3` 的时间格式化可以直接被用来做类似于 [calendar layouts](http://bl.ocks.org/mbostock/4063318) 之类的事情。

## Installing

`NPM` 安装： `npm install d3-time`. 此外还可以下载 [latest release](https://github.com/xswei/d3-time/releases/latest). 可以直接从 [d3js.org](https://d3js.org) 以 [standalone library](https://d3js.org/d3-time.v1.min.js) 或作为 [D3 4.0](https://github.com/xswei/d3) 的一部分引入. 支持 `AMD`, `CommonJS`, 以及基本的标签形式，如果使用标签引入会暴露全局 `d3` 变量:

```html
<script src="https://d3js.org/d3-time.v1.min.js"></script>
<script>

var day = d3.timeDay(new Date);

</script>
```

[在浏览器中测试 `d3-time`.](https://tonicdev.com/npm/d3-time)

## API Reference

<a name="_interval" href="#_interval">#</a> <i>interval</i>(<i>date</i>) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L6 "Source")

[*interval*.floor](#interval_floor) 的别名. 例如 [d3.timeYear](#timeYear)(*date*) 和 d3.timeYear.floor(*date*) 是等价的.

<a name="interval_floor" href="#interval_floor">#</a> <i>interval</i>.<b>floor</b>(<i>date</i>) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L10 "Source")

对日期的向下取整，返回一个新日期，返回的日期不超过当前 *date*。例如 [d3.timeDay](#timeDay).floor(*date*) 通常返回给定 *date* 的当天 `12:00  AM`.

这个方法是幂等的：如果指定的 *date* 在当前时间间隔内，则返回相同时间的新日期。此外返回的日期是当前区间内的最小值，比如 *interval*.floor(*interval*.floor(*date*) - 1) 会返回前一个间隔的最小时间值。

注意 `==` and `===` 操作不能对比 [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) 对象，因此不能使用这个操作来判读指定的 *date* 是否已经完成向下取整。作为替代，你可以将其转换为数值然后对比：

```js
// Returns true if the specified date is a day boundary.
function isDay(date) {
  return +d3.timeDay.floor(date) === +date;
}
```

这比测试时间是否为 `12:00 AM` 更可靠，因为在某些时区，由于夏令时可能不存在午夜。

<a name="interval_round" href="#interval_round">#</a> <i>interval</i>.<b>round</b>(<i>date</i>) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L16 "Source")

日期取整，返回指定 *date* 的取整时刻。例如 [d3.timeDay](#timeDay).round(*date*)，如果当前 *date* 在 `12:00 AM` 之前会返回当天的 `12:00 AM`，否则会返回下一天的 `12:00 AM`.

这个方法也是幂等的：对于一定时间区间内的 *date* 都会返回相同的新时刻。

<a name="interval_ceil" href="#interval_ceil">#</a> <i>interval</i>.<b>ceil</b>(<i>date</i>) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L12 "Source")

日期向上取整，返回不晚于指定 *date* 的最早的时刻。比如，[d3.timeDay](#timeDay).ceil(*date*) 通常会返回指定 *date* 下一天的 `12:00 AM`.

这个方法也是幂等的：对于一定时间区间内的 *date* 都会返回相同的新时刻。因此 *interval*.ceil(*interval*.ceil(*date*) + 1) 会返回当前日期的下下一天的 `12:00 AM`.

<a name="interval_offset" href="#interval_offset">#</a> <i>interval</i>.<b>offset</b>(<i>date</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L22 "Source")

返回一个新的日期，新的日期等于 *date* 加 *step* 间隔。如果没有指定 *step* 则默认为 `1`。如果 *step* 为负，则返回的新日期将比指定的 *date* 早。如果 *step* 为 `0` 则返回指定 *date* 的拷贝；如果 *step* 不是整数，则会使用 [floored](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor) 进行取整。这个方法不会对指定的 *date* 进行四舍五入。例如如果 *date* 为 今天的 `5:34 PM` 则 [d3.timeDay](#timeDay).offset(*date*, 1) 会返回明天的 `5:34 PM`(即使夏令时改变了也能正确返回)。

<a name="interval_range" href="#interval_range">#</a> <i>interval</i>.<b>range</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L26 "Source")

返回介于 *start* (包含) 和 *stop* (不包含) 之间的日期间隔数组。如果 *step* 没有指定则默认返回每个日期间隔边界值；例如 [d3.timeDay](#timeDay) 间隔步长设置为 `2` 时表示每隔一天返回取一个值。如果 *step* 不是整数，则会使用 [floored](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor) 进行取整。

返回的数组的第一个元素的日期值表示所取的日期范围的最早值，不早于 *start*；随后的所有的值之间的间隔都为 *step* 表示的日期间隔。例如只取奇数天：

```js
d3.timeDay.range(new Date(2015, 0, 1), new Date(2015, 0, 7), 2);
// [Thu Jan 01 2015 00:00:00 GMT-0800 (PST),
//  Sat Jan 03 2015 00:00:00 GMT-0800 (PST),
//  Mon Jan 05 2015 00:00:00 GMT-0800 (PST)]
```

只取偶数天：

```js
d3.timeDay.range(new Date(2015, 0, 2), new Date(2015, 0, 8), 2);
// [Fri Jan 02 2015 00:00:00 GMT-0800 (PST),
//  Sun Jan 04 2015 00:00:00 GMT-0800 (PST),
//  Tue Jan 06 2015 00:00:00 GMT-0800 (PST)]
```

如果想要每个间隔的步长一致，可以使用 [*interval*.every](#interval_every) 代替。

<a name="interval_filter" href="#interval_filter">#</a> <i>interval</i>.<b>filter</b>(<i>test</i>) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L35 "Source")

返回一个新的间隔，这个间隔是使用 *test* 函数对指定间隔进行过滤之后的子集。*test* 函数传递日期并且应该返回真或假以表示当前日期是否被被考虑在内。例如创建一个返回每个月 `1st`, `11th`, `21th` and `31th`(如果存在) 的间隔：

```js
var i = d3.timeDay.filter(function(d) { return (d.getDate() - 1) % 10 === 0; });
```

返回的新的间隔不支持 [*interval*.count](#interval_count). 同时参考 [*interval*.every](#interval_every).

<a name="interval_every" href="#interval_every">#</a> <i>interval</i>.<b>every</b>(<i>step</i>) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L50 "Source")

返回当前间隔的经过过滤后的日期列表。*step* 依赖于当前间隔的类型，例如 [d3.timeMinute](#timeMinute).every(15) 会返回一个表示每隔 `15` 分钟取一个值的间隔。需要注意，对于一些间隔返回的日期可能不是均匀的。如果 *step* 不可用则返回 `null`。如果 *step* 为 `1` 则返回当前间隔。

这个方法可以和 [*interval*.range](#interval_range) 结合使用以保障生成固定的日期列表：

```js
d3.timeDay.every(2).range(new Date(2015, 0, 1), new Date(2015, 0, 7));
// [Thu Jan 01 2015 00:00:00 GMT-0800 (PST),
//  Sat Jan 03 2015 00:00:00 GMT-0800 (PST),
//  Mon Jan 05 2015 00:00:00 GMT-0800 (PST)]
```

同上：

```js
d3.timeDay.every(2).range(new Date(2015, 0, 2), new Date(2015, 0, 8));
// [Sat Jan 03 2015 00:00:00 GMT-0800 (PST),
//  Mon Jan 05 2015 00:00:00 GMT-0800 (PST),
//  Wed Jan 07 2015 00:00:00 GMT-0800 (PST)]
```

返回的过滤间隔不支持 [*interval*.count](#interval_count)，同时参考 [*interval*.filter](#interval_filter).

<a name="interval_count" href="#interval_count">#</a> <i>interval</i>.<b>count</b>(<i>start</i>, <i>end</i>) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L44 "Source")

返回以当前间隔为单位，在 *start*(不包含) 之后和 *end*(包含) 之前的个数。注意这个行为与 [*interval*.range](#interval_range) 有所不同，因为它的目标是返回 *end* 相对于 *start* 之间的统计数。例如计算当前日期在当年内的基于 `0` 的天数统计:

```js
var now = new Date;
d3.timeDay.count(d3.timeYear(now), now); // 177
```

同样的，计算当前基于 `0` 的对周日个数统计(过了多少个周日):

```js
d3.timeSunday.count(d3.timeYear(now), now); // 25
```

<a name="timeInterval" href="#timeInterval">#</a> d3.<b>timeInterval</b>(<i>floor</i>, <i>offset</i>[, <i>count</i>[, <i>field</i>]]) [<>](https://github.com/xswei/d3-time/blob/master/src/interval.js#L4 "Source")

构造给定指定的 *floor* 和 *offset* 函数以及可选的 *count* 函数自定义间隔。

*floor* 函数接受单个日期作为参数，并将其四舍五入到最近的区间边界。

*floor* 函数以日期和整数步长作为参数，将指定的日期向前推进指定的边界数; 步长可以是正的，负的或零。

可选的 *count* 函数接收一个起始日期和结束日期，向下取整到当前间隔，并返回开始(独占)和结束(包含)之间的边界数。

The optional *count* function takes a start date and an end date, already floored to the current interval, and returns the number of boundaries between the start (exclusive) and end (inclusive). If a *count* function is not specified, the returned interval does not expose [*interval*.count](#interval_count) or [*interval*.every](#interval_every) methods. Note: due to an internal optimization, the specified *count* function must not invoke *interval*.count on other time intervals.

The optional *field* function takes a date, already floored to the current interval, and returns the field value of the specified date, corresponding to the number of boundaries between this date (exclusive) and the latest previous parent boundary. For example, for the [d3.timeDay](#timeDay) interval, this returns the number of days since the start of the month. If a *field* function is not specified, it defaults to counting the number of interval boundaries since the UNIX epoch of January 1, 1970 UTC. The *field* function defines the behavior of [*interval*.every](#interval_every).

### Intervals

The following intervals are provided:

<a name="timeMillisecond" href="#timeMillisecond">#</a> d3.<b>timeMillisecond</b> [<>](https://github.com/xswei/d3-time/blob/master/src/millisecond.js "Source")
<br><a href="#timeMillisecond">#</a> d3.<b>utcMillisecond</b>

Milliseconds; the shortest available time unit.

<a name="timeSecond" href="#timeSecond">#</a> d3.<b>timeSecond</b> [<>](https://github.com/xswei/d3-time/blob/master/src/second.js "Source")
<br><a href="#timeSecond">#</a> d3.<b>utcSecond</b>

Seconds (e.g., 01:23:45.0000 AM); 1,000 milliseconds.

<a name="timeMinute" href="#timeMinute">#</a> d3.<b>timeMinute</b> [<>](https://github.com/xswei/d3-time/blob/master/src/minute.js "Source")
<br><a href="#timeMinute">#</a> d3.<b>utcMinute</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcMinute.js "Source")

Minutes (e.g., 01:02:00 AM); 60 seconds. Note that ECMAScript [ignores leap seconds](http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.1).

<a name="timeHour" href="#timeHour">#</a> d3.<b>timeHour</b> [<>](https://github.com/xswei/d3-time/blob/master/src/hour.js "Source")
<br><a href="#timeHour">#</a> d3.<b>utcHour</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcHour.js "Source")

Hours (e.g., 01:00 AM); 60 minutes. Note that advancing time by one hour in local time can return the same hour or skip an hour due to daylight saving.

<a name="timeDay" href="#timeDay">#</a> d3.<b>timeDay</b> [<>](https://github.com/xswei/d3-time/blob/master/src/day.js "Source")
<br><a href="#timeDay">#</a> d3.<b>utcDay</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcDay.js "Source")

Days (e.g., February 7, 2012 at 12:00 AM); typically 24 hours. Days in local time may range from 23 to 25 hours due to daylight saving.

<a name="timeWeek" href="#timeWeek">#</a> d3.<b>timeWeek</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js "Source")
<br><a href="#timeWeek">#</a> d3.<b>utcWeek</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js "Source")

Alias for [d3.timeSunday](#timeSunday); 7 days and typically 168 hours. Weeks in local time may range from 167 to 169 hours due on daylight saving.

<a name="timeSunday" href="#timeSunday">#</a> d3.<b>timeSunday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L15 "Source")
<br><a href="#timeSunday">#</a> d3.<b>utcSunday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L15 "Source")

Sunday-based weeks (e.g., February 5, 2012 at 12:00 AM).

<a name="timeMonday" href="#timeMonday">#</a> d3.<b>timeMonday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L16 "Source")
<br><a href="#timeMonday">#</a> d3.<b>utcMonday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L16 "Source")

Monday-based weeks (e.g., February 6, 2012 at 12:00 AM).

<a name="timeTuesday" href="#timeTuesday">#</a> d3.<b>timeTuesday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L17 "Source")
<br><a href="#timeTuesday">#</a> d3.<b>utcTuesday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L17 "Source")

Tuesday-based weeks (e.g., February 7, 2012 at 12:00 AM).

<a name="timeWednesday" href="#timeWednesday">#</a> d3.<b>timeWednesday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L18 "Source")
<br><a href="#timeWednesday">#</a> d3.<b>utcWednesday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L18 "Source")

Wednesday-based weeks (e.g., February 8, 2012 at 12:00 AM).

<a name="timeThursday" href="#timeThursday">#</a> d3.<b>timeThursday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L19 "Source")
<br><a href="#timeThursday">#</a> d3.<b>utcThursday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L19 "Source")

Thursday-based weeks (e.g., February 9, 2012 at 12:00 AM).

<a name="timeFriday" href="#timeFriday">#</a> d3.<b>timeFriday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L20 "Source")
<br><a href="#timeFriday">#</a> d3.<b>utcFriday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L20 "Source")

Friday-based weeks (e.g., February 10, 2012 at 12:00 AM).

<a name="timeSaturday" href="#timeSaturday">#</a> d3.<b>timeSaturday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L21 "Source")
<br><a href="#timeSaturday">#</a> d3.<b>utcSaturday</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L21 "Source")

Saturday-based weeks (e.g., February 11, 2012 at 12:00 AM).

<a name="timeMonth" href="#timeMonth">#</a> d3.<b>timeMonth</b> [<>](https://github.com/xswei/d3-time/blob/master/src/month.js "Source")
<br><a href="#timeMonth">#</a> d3.<b>utcMonth</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcMonth.js "Source")

Months (e.g., February 1, 2012 at 12:00 AM); ranges from 28 to 31 days.

<a name="timeYear" href="#timeYear">#</a> d3.<b>timeYear</b> [<>](https://github.com/xswei/d3-time/blob/master/src/year.js "Source")
<br><a href="#timeYear">#</a> d3.<b>utcYear</b> [<>](https://github.com/xswei/d3-time/blob/master/src/utcYear.js "Source")

Years (e.g., January 1, 2012 at 12:00 AM); ranges from 365 to 366 days.

### Ranges

For convenience, aliases for [*interval*.range](#interval_range) are also provided as plural forms of the corresponding interval.

<a name="timeMilliseconds" href="#timeMilliseconds">#</a> d3.<b>timeMilliseconds</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/millisecond.js#L26 "Source")
<br><a href="#timeMilliseconds">#</a> d3.<b>utcMilliseconds</b>(<i>start</i>, <i>stop</i>[, <i>step</i>])

Aliases for [d3.timeMillisecond](#timeMillisecond).[range](#interval_range) and [d3.utcMillisecond](#timeMillisecond).[range](#interval_range).

<a name="timeSeconds" href="#timeSeconds">#</a> d3.<b>timeSeconds</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/second.js#L15 "Source")
<br><a href="#timeSeconds">#</a> d3.<b>utcSeconds</b>(<i>start</i>, <i>stop</i>[, <i>step</i>])

Aliases for [d3.timeSecond](#timeSecond).[range](#interval_range) and [d3.utcSecond](#timeSecond).[range](#interval_range).

<a name="timeMinutes" href="#timeMinutes">#</a> d3.<b>timeMinutes</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/minute.js#L15 "Source")
<br><a href="#timeMinutes">#</a> d3.<b>utcMinutes</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcMinute.js#L15 "Source")

Aliases for [d3.timeMinute](#timeMinute).[range](#interval_range) and [d3.utcMinute](#timeMinute).[range](#interval_range).

<a name="timeHours" href="#timeHours">#</a> d3.<b>timeHours</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/hour.js#L17 "Source")
<br><a href="#timeHours">#</a> d3.<b>utcHours</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcHour.js#L15 "Source")

Aliases for [d3.timeHour](#timeHour).[range](#interval_range) and [d3.utcHour](#timeHour).[range](#interval_range).

<a name="timeDays" href="#timeDays">#</a> d3.<b>timeDays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/day.js#L15 "Source")
<br><a href="#timeDays">#</a> d3.<b>utcDays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcDay.js#L15 "Source")

Aliases for [d3.timeDay](#timeDay).[range](#interval_range) and [d3.utcDay](#timeDay).[range](#interval_range).

<a name="timeWeeks" href="#timeWeeks">#</a> d3.<b>timeWeeks</b>(<i>start</i>, <i>stop</i>[, <i>step</i>])
<br><a href="#timeWeeks">#</a> d3.<b>utcWeeks</b>(<i>start</i>, <i>stop</i>[, <i>step</i>])

Aliases for [d3.timeWeek](#timeWeek).[range](#interval_range) and [d3.utcWeek](#timeWeek).[range](#interval_range).

<a name="timeSundays" href="#timeSundays">#</a> d3.<b>timeSundays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L23 "Source")
<br><a href="#timeSundays">#</a> d3.<b>utcSundays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L23 "Source")

Aliases for [d3.timeSunday](#timeSunday).[range](#interval_range) and [d3.utcSunday](#timeSunday).[range](#interval_range).

<a name="timeMondays" href="#timeMondays">#</a> d3.<b>timeMondays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L24 "Source")
<br><a href="#timeMondays">#</a> d3.<b>utcMondays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L24 "Source")

Aliases for [d3.timeMonday](#timeMonday).[range](#interval_range) and [d3.utcMonday](#timeMonday).[range](#interval_range).

<a name="timeTuesdays" href="#timeTuesdays">#</a> d3.<b>timeTuesdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L25 "Source")
<br><a href="#timeTuesdays">#</a> d3.<b>utcTuesdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L25 "Source")

Aliases for [d3.timeTuesday](#timeTuesday).[range](#interval_range) and [d3.utcTuesday](#timeTuesday).[range](#interval_range).

<a name="timeWednesdays" href="#timeWednesdays">#</a> d3.<b>timeWednesdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L26 "Source")
<br><a href="#timeWednesdays">#</a> d3.<b>utcWednesdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L26 "Source")

Aliases for [d3.timeWednesday](#timeWednesday).[range](#interval_range) and [d3.utcWednesday](#timeWednesday).[range](#interval_range).

<a name="timeThursdays" href="#timeThursdays">#</a> d3.<b>timeThursdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L27 "Source")
<br><a href="#timeThursdays">#</a> d3.<b>utcThursdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L27 "Source")

Aliases for [d3.timeThursday](#timeThursday).[range](#interval_range) and [d3.utcThursday](#timeThursday).[range](#interval_range).

<a name="timeFridays" href="#timeFridays">#</a> d3.<b>timeFridays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L28 "Source")
<br><a href="#timeFridays">#</a> d3.<b>utcFridays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L28 "Source")

Aliases for [d3.timeFriday](#timeFriday).[range](#interval_range) and [d3.utcFriday](#timeFriday).[range](#interval_range).

<a name="timeSaturdays" href="#timeSaturdays">#</a> d3.<b>timeSaturdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/week.js#L29 "Source")
<br><a href="#timeSaturdays">#</a> d3.<b>utcSaturdays</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcWeek.js#L29 "Source")

Aliases for [d3.timeSaturday](#timeSaturday).[range](#interval_range) and [d3.utcSaturday](#timeSaturday).[range](#interval_range).

<a name="timeMonths" href="#timeMonths">#</a> d3.<b>timeMonths</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/month.js#L15 "Source")
<br><a href="#timeMonths">#</a> d3.<b>utcMonths</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcMonth.js#L15 "Source")

Aliases for [d3.timeMonth](#timeMonth).[range](#interval_range) and [d3.utcMonth](#timeMonth).[range](#interval_range).

<a name="timeYears" href="#timeYears">#</a> d3.<b>timeYears</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/year.js#L26 "Source")
<br><a href="#timeYears">#</a> d3.<b>utcYears</b>(<i>start</i>, <i>stop</i>[, <i>step</i>]) [<>](https://github.com/xswei/d3-time/blob/master/src/utcYear.js#L26 "Source")

Aliases for [d3.timeYear](#timeYear).[range](#interval_range) and [d3.utcYear](#timeYear).[range](#interval_range).
