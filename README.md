# vue2-calendar

> ??Vue2?????

<p align="center">
  <a href="http://blog.w3cub.com/vue2-calendar/?x-page=github">
    Live Demo >>
  </a>
  <br>
  <br>
  <a href="https://github.com/icai/vue2-calendar">
    <img src="https://img.shields.io/github/stars/icai/vue2-calendar.svg?style=social&label=Star" alt="">
  </a>
  <a href="https://github.com/icai/vue2-calendar">
    <img src="https://img.shields.io/github/forks/icai/vue2-calendar.svg?style=social&label=Fork" alt="">
  </a>
  <a href="https://github.com/icai/vue2-calendar">
    <img src="https://img.shields.io/github/watchers/icai/vue2-calendar.svg?style=social&label=Watch" alt="">
  </a>
  <br>
  <br>
  <a href="https://github.com/icai/vue2-calendar/issues">
    <img src="https://img.shields.io/github/issues/icai/vue2-calendar.svg?style=flat-square" alt="">
  </a>
  <a href="https://github.com/icai/vue2-calendar/issues">
    <img src="http://isitmaintained.com/badge/resolution/icai/vue2-calendar.svg?style=flat-square" alt="">
  </a>
  <a href="https://github.com/icai/vue2-calendar/graphs/contributors">
    <img src="https://img.shields.io/github/contributors/icai/vue2-calendar.svg?style=flat-square" alt="">
  </a>
  <br>
  <a href="https://www.npmjs.com/package/vue2-slot-calendar">
    <img src="https://img.shields.io/npm/l/vue2-slot-calendar.svg?style=flat-square" alt="">
  </a>
  <a href="https://www.npmjs.com/package/vue2-slot-calendar">
    <img src="https://img.shields.io/npm/v/vue2-slot-calendar.svg?style=flat-square" alt="">
  </a>
  <a href="https://www.npmjs.com/package/vue2-slot-calendar">
    <img src="https://img.shields.io/npm/dm/vue2-slot-calendar.svg?style=flat-square" alt="">
  </a>
  <a href="https://www.npmjs.com/package/vue2-slot-calendar">
    <img src="https://img.shields.io/npm/dt/vue2-slot-calendar.svg?style=flat-square" alt="">
  </a>
  <br>
  <br>
</p>

- This project is not only a vue component, but also a webpack **multi-page** project in action.

- Boostrap style like

- I18n support

- Community feedback


<h2>?????node???</h2>
<div class="cnblogs_Highlighter">
<pre class="brush:bash;gutter:true;">$ npm install vue2-slot-calendar
</pre>
</div>
<h3>????</h3>
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
<h3>????????js??????</h3>
<div class="cnblogs_Highlighter">
<pre class="brush:html;gutter:true;">&lt;link rel="stylesheet" href="/lib/calendar.min.css" &gt;
&lt;script src="/lib/calendar.min.js"&gt;&lt;/script&gt;
</pre>
</div>
<h2>?????</h2>
<p>????<span style="background-color: #ffff00; color: #0000ff;">window.VueCalendarLang </span>??????????????</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">window.VueCalendarLang=function(lang){
    return {}; //???????JSON?? 
};
</pre>
</div>
<p>?????????????????JSON???</p>
<p>?????? /src/lang/zh-CN.js ??</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">{
    daysOfWeek: ["?", "?", "?", "?", "?", "?", "?"],
    limit: "???? (??{{limit}}?)",
    loading: "???...",
    minLength: "????",
    months: [
        "??",
        "??",
        "??",
        "??",
        "??",
        "??",
        "??",
        "??",
        "??",
        "??",
        "??",
        "??"
    ],
    notSelected: "???",
    required: "???",
    search: "??"
}
</pre>
</div>
<p>??</p>
<h2>????</h2>
<div class="cnblogs_Highlighter">
<pre class="brush:bash;gutter:true;"># ????
npm install

# ??????? localhost:4000
npm run dev

# ?????JS?CSS??
npm run build

# ????
npm run unit

# ??????
npm test
</pre>
</div>
<h3>?????</h3>
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
<td>?????????? v-model ??</td>
</tr>
<tr>
<td>width</td>
<td><code>String</code></td>
<td>'200px'</td>
<td>input??????</td>
</tr>
<tr>
<td>format</td>
<td><code>String</code></td>
<td><code>yyyy-MM-dd</code></td>
<td>????,???? d, dd, M, MM, MMM, MMMM, yyyy.</td>
</tr>
<tr>
<td>disabled-days-of-week</td>
<td><code>Array</code></td>
<td>&nbsp;</td>
<td>????????????0-6??????????</td>
</tr>
<tr>
<td>clear-button</td>
<td><code>Bollean</code></td>
<td>false</td>
<td>??????????????????????????????????????????</td>
</tr>
<tr>
<td>placeholder</td>
<td><code>String</code></td>
<td>&nbsp;</td>
<td>?????????</td>
</tr>
<tr>
<td>hasInput</td>
<td><code>Boolean</code></td>
<td>true</td>
<td>???????</td>
</tr>
<tr>
<td>pane</td>
<td><code>Number</code></td>
<td>1</td>
<td>?????????1???????2????????1?2</td>
</tr>
<tr>
<td>borderWidth</td>
<td><code>Number</code></td>
<td>2</td>
<td>?????????</td>
</tr>
<tr>
<td><span style="color: #0000ff;">onDayClick</span></td>
<td><span style="color: #0000ff;"><code>Function</code></span></td>
<td><span style="color: #0000ff;">&nbsp;</span></td>
<td><span style="color: #0000ff;">??????????????hasInput?????????</span></td>
</tr>
<tr>
<td>specialDays</td>
<td><code>Object</code></td>
<td>&nbsp;</td>
<td>????</td>
</tr>
<tr>
<td>changePane</td>
<td><code>Function</code></td>
<td>&nbsp;</td>
<td>??????????, ??????????????????????&nbsp;&nbsp;<code>/src/modules/Docs.vue</code></td>
</tr>
<tr>
<td>rangeBus</td>
<td><code>Function</code></td>
<td>&nbsp;</td>
<td>???????????????? new Vue() ???</td>
</tr>
<tr>
<td>rangeStatus</td>
<td><code>Number</code></td>
<td>0</td>
<td>?????????0??????????1?????????????2????????????</td>
</tr>
<tr>
<td>onDrawDate</td>
<td><code>Function</code></td>
<td>&nbsp;</td>
<td>?????????????????????</td>
</tr>
<tr>
<td>showDateOnly</td>
<td><code>Boolean</code></td>
<td>false</td>
<td>?????????</td>
</tr>
<tr>
<td>transfer</td>
<td><code>Boolean</code></td>
<td>false</td>
<td>?????????? document.body?</td>
</tr>
<tr>
<td>elementId</td>
<td><code>String</code></td>
<td>&nbsp;</td>
<td>??ID</td>
</tr>
<tr>
<td>firstDayOfWeek</td>
<td><code>Number</code></td>
<td>0</td>
<td>???1??0??????1????????????</td>
</tr>
</tbody>
</table>
<h3 class="prettyprint prettyprinted"><br />????????</h3>
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
