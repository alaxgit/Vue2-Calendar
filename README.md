# vue2-calendar

> 基于vue2.js 和 bootstrap.css 样式的日历控件。
- 本人于2019.03.29在原 v2.2.4 版基础上作了优化，因此改版本号为 v2.2.5。
- 改动内容：
-   1.改进原控件无法切换语言的BUG，支持 lang='zh-CN'和'en'。
-   2.日历面板增加一个位置选项，允许让面板右对齐，只需要position=right即可。
-   3.原 :on-day-click 事件只有在 hasInput=false 时才生效，现改为一直有效，这样当用户点击日历面板上的日期后就可以触发该事件。
-   4.clear=false时文本框右侧会显示一个日历图标，高仿bootstrap的日历组件，但clear=true时不显示


- 这不仅是一个Vue组件，更是一个多页应用项目

- Boostrap风格

- 多语言支持


<h2>安装（需要node支持）</h2>
<div class="cnblogs_Highlighter">
<pre class="brush:bash;gutter:true;">$ npm install vue2-slot-calendar
</pre>
</div>
<h3>导入模块</h3>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">// js file
import 'vue2-slot-calendar/lib/calendar.min.css';
import calendar from 'vue2-slot-calendar/lib/calendar';

// vue file
// in ES6 modules
import Calendar from 'vue2-slot-calendar';

// in CommonJS
const Calendar = require('vue2-slot-calendar');

// in Global variable
const VueCalendar = Calendar;
</pre>
</div>
<h3>直接引用编译好的js文件（推荐）</h3>
<div class="cnblogs_Highlighter">
<pre class="brush:html;gutter:true;">&lt;link rel="stylesheet" href="/lib/calendar.min.css" &gt;
&lt;script src="/lib/calendar.min.js"&gt;&lt;/script&gt;
</pre>
</div>
<h2>多语言支持</h2>
<p>系统提供<span style="background-color: #ffff00; color: #0000ff;">window.VueCalendarLang </span>勾子函数来让用户自定义语言，</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">window.VueCalendarLang=function(lang){
    return {}; //对象必须是下面JSON格式 
};
</pre>
</div>
<p>上面勾子函数返回的必须是下面格式的JSON对象。</p>
<p>下面的脚本在 /src/lang/zh-CN.js 中有</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">{
    daysOfWeek: ["日", "一", "二", "三", "四", "五", "六"],
    limit: "超过限制 (最多{{limit}}项)",
    loading: "加载中...",
    minLength: "最小长度",
    months: [
        "一月",
        "二月",
        "三月",
        "四月",
        "五月",
        "六月",
        "七月",
        "八月",
        "九月",
        "十月",
        "十一",
        "十二"
    ],
    notSelected: "未选择",
    required: "必填项",
    search: "查找"
}
</pre>
</div>
<p>　　</p>
<h2>构建步骤</h2>
<div class="cnblogs_Highlighter">
<pre class="brush:bash;gutter:true;"># 安装依赖
npm install

# 运行开发服务器 localhost:4000
npm run dev

# 编译（压缩JS和CSS等）
npm run build

# 单元测试
npm run unit

# 运行所有测试
npm test
</pre>
</div>
<h3>参数和事件</h3>
<p>&nbsp;</p>
<table style="height: 447px; width: 1092px;">
<thead>
<tr><th>Name</th><th>Type</th><th>Default</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td>value</td>
<td><code>String</code></td>
<td>''</td>
<td>组件输入输出的值，即 v-model 的值</td>
</tr>
<tr>
<td>width</td>
<td><code>String</code></td>
<td>'200px'</td>
<td>input文本框的宽度</td>
</tr>
<tr>
<td>format</td>
<td><code>String</code></td>
<td><code>yyyy-MM-dd</code></td>
<td>日期格式,可用值： d, dd, M, MM, MMM, MMMM, yyyy.</td>
</tr>
<tr>
<td>disabled-days-of-week</td>
<td><code>Array</code></td>
<td>&nbsp;</td>
<td>每周有哪些天禁用，可用值0-6，多个值则用逗号隔开</td>
</tr>
<tr>
<td>clear-button</td>
<td><code>Bollean</code></td>
<td>false</td>
<td>是否显示清除按钮，显示清除按钮时右侧的日历图标不显示，不显示清除按钮时则显示日历图标</td>
</tr>
<tr>
<td>placeholder</td>
<td><code>String</code></td>
<td>&nbsp;</td>
<td>文本框中显示的水印</td>
</tr>
<tr>
<td>hasInput</td>
<td><code>Boolean</code></td>
<td>true</td>
<td>是否显示文本框</td>
</tr>
<tr>
<td>pane</td>
<td><code>Number</code></td>
<td>1</td>
<td>日历面板数量，默认1，双月日历设为2即可，目前仅支持1和2</td>
</tr>
<tr>
<td>borderWidth</td>
<td><code>Number</code></td>
<td>2</td>
<td>日历面板边框线宽度</td>
</tr>
<tr>
<td><span style="color: #0000ff;">onDayClick</span></td>
<td><span style="color: #0000ff;"><code>Function</code></span></td>
<td><span style="color: #0000ff;">&nbsp;</span></td>
<td><span style="color: #0000ff;">点击日期时的事件，已改为不受hasInput值的限制，总是触发</span></td>
</tr>
<tr>
<td>specialDays</td>
<td><code>Object</code></td>
<td>&nbsp;</td>
<td>特殊日期</td>
</tr>
<tr>
<td>changePane</td>
<td><code>Function</code></td>
<td>&nbsp;</td>
<td>切换日历面板时的事件, 暂时还没研究这个，大多数情况下用不到，请参考&nbsp;&nbsp;<code>/src/modules/Docs.vue</code></td>
</tr>
<tr>
<td>rangeBus</td>
<td><code>Function</code></td>
<td>&nbsp;</td>
<td>暂时不明白啥意思，请总是返回一个 new Vue() 就对了</td>
</tr>
<tr>
<td>rangeStatus</td>
<td><code>Number</code></td>
<td>0</td>
<td>日期范围状态，默认0表示不使用日期范围，1表示双日历联动的起始日期，2表示双日历联动的结束日期</td>
</tr>
<tr>
<td>onDrawDate</td>
<td><code>Function</code></td>
<td>&nbsp;</td>
<td>绘制日期时的事件，可以让用户自定义日历样式</td>
</tr>
<tr>
<td>showDateOnly</td>
<td><code>Boolean</code></td>
<td>false</td>
<td>是否只显示日历面板</td>
</tr>
<tr>
<td>transfer</td>
<td><code>Boolean</code></td>
<td>false</td>
<td>是否将日历面板添加到 document.body中</td>
</tr>
<tr>
<td>elementId</td>
<td><code>String</code></td>
<td>&nbsp;</td>
<td>日历ID</td>
</tr>
<tr>
<td>firstDayOfWeek</td>
<td><code>Number</code></td>
<td>0</td>
<td>每周第1天，0表示星期天，1表示星期一，其它依次类推</td>
</tr>
</tbody>
</table>
<h3 class="prettyprint prettyprinted"><br />组件的所有属性值</h3>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">props: {
    value: {
      type: [String, Date]
    },
    format: {
      default: 'yyyy-MM-dd'
    },
    firstDayOfWeek: {
      // sunday
      default: 0
    },
    disabledDaysOfWeek: {
      type: Array,
      default () {
        return []
      }
    },
    width: {
      type: String,
      default: '200px'
    },
    clearButton: {
      type: Boolean,
      default: false
    },
    inputClasses: {
      type: String,
      default: ''
    },
    lang: {
      type: String,
      default: navigator.language
    },
    placeholder: {
      type: String
    },
    hasInput: {
      type: Boolean,
      default: true
    },
    pane: {
      type: Number,
      default: 1
    },
    borderWidth: {
      type: Number,
      default: 2
    },
    onDayClick: {
      type: Function,
      default () {}
    },
    changePane: {
      type: Function,
      default () {}
    },
    specialDays: {
      type: Object,
      default () {
        return {}
      }
    },
    rangeBus: {
      type: Function,
      default () {
        // return new Vue()
      }
    },
    rangeStatus: {
      type: Number,
      default: 0
    },
    onDrawDate: {
      type: Function,
      default () {}
    },
    maxDate: {
      type: String
    },
    minDate: {
      type: String
    },
    showDateOnly: {
      type: Boolean,
      default: false
    },
    transfer: {
      type: Boolean,
      default: false
    },
    elementId: [String]
  }
</pre>
</div>
## Credits

Inspired by [vue-strap](https://github.com/yuche/vue-strap) datepicker component.

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
