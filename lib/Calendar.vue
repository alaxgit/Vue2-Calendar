<template>
  <div class="datepicker">
    <template v-if="hasInput">
      <input
        :id="elementId"
        class="form-control datepicker-input"
        :class="classes"
        type="text"
        :placeholder="placeholder"
        :style="{width:width}"
        @click="inputClick"
        v-model="inputValue"
        readonly="readonly"
      >
      <button v-if="clearButton && value" type="button" class="close" @click="inputValue = ''">
        <span>&times;</span>
      </button>
      <button v-else type="button" class="datepicker-input-icon" @click="inputClick">
        <span class="calendaricon-calendar"></span>
      </button>
    </template>
    <div
      class="datepicker-wrapper"
      ref="popup"
      v-transfer="transfer"
      v-show="isWrapperShow"
      :style="paneStyle"
      :class="panePosition"
    >
      <div
        class="datepicker-popup"
        @mouseover="handleMouseOver"
        @mouseout="handleMouseOver"
        v-show="displayDayView"
      >
        <div class="datepicker-ctrl">
          <span
            class="datepicker-preBtn calendaricon-angle-left"
            aria-hidden="true"
            @click="preNextMonthClick(0)"
          ></span>
          <span
            class="datepicker-nextBtn calendaricon-angle-right"
            aria-hidden="true"
            @click="preNextMonthClick(1)"
          ></span>
        </div>
        <template v-for="(p, pan) in pane">
          <div class="datepicker-inner" :key="pan">
            <div class="datepicker-body">
              <p @click="switchMonthView">{{stringifyDayHeader(currDate, pan)}}</p>
              <div class="datepicker-weekRange">
                <span v-for="(w, index) in daysOfWeek" :key="index">{{w}}</span>
              </div>
              <div class="datepicker-dateRange">
                <span
                  v-for="(d, k) in dateRange[pan]"
                  :key="k"
                  class="day-cell"
                  :class="getItemClasses(d)"
                  :data-date="stringify(d.date)"
                  @click="daySelect(d, $event)"
                >
                  <div>
                    <template
                      v-if="d.sclass !== 'datepicker-item-gray'"
                    >{{getSpecailDay(d.date) || d.text}}</template>
                    <template v-else>{{d.text}}</template>
                    <div v-if="d.sclass !== 'datepicker-item-gray'">
                      <slot :name="stringify(d.date)"></slot>
                    </div>
                  </div>
                </span>
              </div>
            </div>
          </div>
        </template>
      </div>
      <div class="datepicker-popup" v-if="!showDateOnly" v-show="displayMonthView">
        <div class="datepicker-ctrl">
          <span
            class="datepicker-preBtn calendaricon-angle-left"
            aria-hidden="true"
            @click="preNextYearClick(0)"
          ></span>
          <span
            class="datepicker-nextBtn calendaricon-angle-right"
            aria-hidden="true"
            @click="preNextYearClick(1)"
          ></span>
        </div>
        <template v-for="(p, pan) in pane">
          <div class="datepicker-inner" :key="pan">
            <div class="datepicker-body">
              <p @click="switchDecadeView">{{stringifyYearHeader(currDate, pan)}}</p>
              <div class="datepicker-monthRange">
                <template v-for="(m, $index) in text.months">
                  <span
                    :key="$index"
                    :class="{'datepicker-dateRange-item-active':
                      (text.months[parse(value).getMonth()]  === m) &&
                      currDate.getFullYear() + pan === parse(value).getFullYear()}"
                    @click="monthSelect(stringifyYearHeader(currDate, pan), $index)"
                  >{{m.substr(0,3)}}</span>
                </template>
              </div>
            </div>
          </div>
        </template>
      </div>
      <div class="datepicker-popup" v-if="!showDateOnly" v-show="displayYearView">
        <div class="datepicker-ctrl">
          <span
            class="datepicker-preBtn calendaricon-angle-left"
            aria-hidden="true"
            @click="preNextDecadeClick(0)"
          ></span>
          <span
            class="datepicker-nextBtn calendaricon-angle-right"
            aria-hidden="true"
            @click="preNextDecadeClick(1)"
          ></span>
        </div>
        <template v-for="(p, pan) in pane">
          <div class="datepicker-inner" :key="pan">
            <div class="datepicker-body">
              <p>{{stringifyDecadeHeader(currDate, pan)}}</p>
              <div class="datepicker-monthRange decadeRange">
                <template v-for="(decade, di) in decadeRange[pan]">
                  <span
                    :key="di"
                    :class="{'datepicker-dateRange-item-active':
                      parse(inputValue).getFullYear() === decade.text}"
                    @click.stop="yearSelect(decade.text)"
                  >{{decade.text}}</span>
                </template>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import LANG_ZH_CN from "../lang/zh-CN";
import LANG_ZH_TW from "../lang/zh-TW";
import LANG_EN from "../lang/en";

function getTarget(node) {
  if (node === void 0) {
    node = document.body;
  }
  if (node === true) {
    return document.body;
  }
  return node instanceof window.Node ? node : document.querySelector(node);
}

const transfer = {
  inserted(el, binding) {
    const value = binding.value;
    if (value === false) return false;
    el.className = el.className ? el.className + " v-transfer" : "v-transfer";
    const parentNode = el.parentNode;
    if (!parentNode) return;
    const home = document.createComment("");
    let hasMovedOut = false;
    if (value !== false) {
      parentNode.replaceChild(home, el); // moving out, el is no longer in the document
      getTarget(value).appendChild(el); // moving into new place
      hasMovedOut = true;
    }
    if (!el.__transferDomData) {
      el.__transferDomData = {
        parentNode: parentNode,
        home: home,
        target: getTarget(value),
        hasMovedOut: hasMovedOut
      };
    }
  },
  componentUpdated(el, binding) {
    const value = binding.value;
    if (value === false) return false;
    // need to make sure children are done updating (vs. `update`)
    const ref$1 = el.__transferDomData;
    if (!ref$1) return;
    // homes.get(el)
    const parentNode = ref$1.parentNode;
    const home = ref$1.home;
    const hasMovedOut = ref$1.hasMovedOut; // recall where home is

    if (!hasMovedOut && value) {
      // remove from document and leave placeholder
      parentNode.replaceChild(home, el);
      // append to target
      getTarget(value).appendChild(el);
      el.__transferDomData = Object.assign({}, el.__transferDomData, {
        hasMovedOut: true,
        target: getTarget(value)
      });
    } else if (hasMovedOut && value === false) {
      // previously moved, coming back home
      parentNode.replaceChild(el, home);
      el.__transferDomData = Object.assign({}, el.__transferDomData, {
        hasMovedOut: false,
        target: getTarget(value)
      });
    } else if (value) {
      // already moved, going somewhere else
      getTarget(value).appendChild(el);
    }
  },
  unbind(el, binding) {
    const value = binding.value;
    if (value === false) return false;
    el.className = el.className.replace("v-transfer", "");
    const ref$1 = el.__transferDomData;
    if (!ref$1) return;
    if (el.__transferDomData.hasMovedOut === true) {
      el.__transferDomData.parentNode &&
        el.__transferDomData.parentNode.appendChild(el);
    }
    el.__transferDomData = null;
  }
};
export default {
  name: "calendar",
  props: {
    value: {
      type: [String, Date]
    },
    format: {
      default: "MM/dd/yyyy"
    },
    firstDayOfWeek: {
      // sunday
      default: 0
    },
    disabledDaysOfWeek: {
      type: Array,
      default() {
        return [];
      }
    },
    width: {
      type: String,
      default: "200px"
    },
    position: {
      type: String,
      default: "left"
    },
    clearButton: {
      type: Boolean,
      default: false
    },
    inputClasses: {
      type: String,
      default: ""
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
      default() {}
    },
    changePane: {
      type: Function,
      default() {}
    },
    specialDays: {
      type: Object,
      default() {
        return {};
      }
    },
    rangeBus: {
      type: Function,
      default() {
        // return new Vue()
      }
    },
    rangeStatus: {
      type: Number,
      default: 0
    },
    onDrawDate: {
      type: Function,
      default() {}
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
  },
  directives: {
    transfer
  },
  mounted() {
    this._blur = e => {
      if (this.transfer) {
        if (
          !this.$el.contains(e.target) &&
          !this.$refs.popup.contains(e.target) &&
          this.hasInput
        )
          this.close();
      } else {
        if (!this.$el.contains(e.target) && this.hasInput) this.close();
      }
    };
    this.$emit("child-created", this);
    // this.inputValue = this.value
    // this.dateFormat = this.format
    this.currDate = this.parse(this.inputValue) || this.parse(new Date());
    const year = this.currDate.getFullYear();
    const month = this.currDate.getMonth();
    this.changePane(year, month, this.pane);
    if (!this.hasInput) {
      this.displayDayView = true;
      this.updatePaneStyle();
    }
    if (this.rangeStatus) {
      this.eventbus = this.rangeBus();
      if (typeof this.eventbus === "object" && !this.eventbus.$on) {
        console.warn("Calendar rangeBus doesn't exist");
        this.rangeStatus = 0;
      }
    }
    if (this.rangeStatus === 2) {
      this._updateRangeStart = date => {
        this.rangeStart = date;
        this.currDate = date;
        this.inputValue = this.stringify(this.currDate);
      };
      this.eventbus.$on("calendar-rangestart", this._updateRangeStart);
    }
    document.addEventListener("click", this._blur);
  },
  beforeDestroy() {
    document.removeEventListener("click", this._blur);
    if (this.rangeStatus === 2) {
      this.eventbus.$off("calendar-rangestart", this._updateRangeStart);
    }
  },
  data() {
    return {
      // inputValue: this.value,
      dateFormat: this.format,
      currDate: new Date(),
      dateRange: [],
      decadeRange: [],
      paneStyle: {
        width: ""
      },
      displayDayView: false,
      displayMonthView: false,
      displayYearView: false,
      rangeStart: false,
      rangeEnd: false
    };
  },
  watch: {
    currDate() {
      this.getDateRange();
    },
    value(v) {
      this.inputValue = v instanceof Date ? this.stringify(v) : v;
    }
  },
  computed: {
    // get real order of the daysOfWeek
    daysOfWeek() {
      const firstDay = this.firstDayOfWeek;
      if (firstDay == 0) {
        return this.text.daysOfWeek;
      }
      return this.text.daysOfWeek
        .slice(firstDay, 7)
        .concat(this.text.daysOfWeek.slice(0, firstDay));
    },
    text() {
      return this.translations(this.lang);
    },
    isWrapperShow() {
      return (
        this.displayDayView || this.displayMonthView || this.displayYearView
      );
    },
    inputValue: {
      get() {
        if (this.value instanceof Date) {
          return this.stringify(this.value);
        } else {
          return this.value;
        }
      },
      set(v) {
        this.$emit("input", v);
        this.currDate = this.parse(v);
        if (this.rangeStatus === 1 && this.eventbus) {
          this.eventbus.$emit("calendar-rangestart", this.currDate);
        }
      }
    },
    classes() {
      var classes = this.inputClasses;
      if (this.clearButton) {
        classes += " with-reset-button";
      }
      return classes;
    },
    panePosition: {
      get() {
        if (this.position == "right") {
          return "datepicker-wrapper-right";
        }
        return "";
      }
    }
  },
  methods: {
    handleMouseOver(event) {
      let target = event.target;
      // this.rangeEnd = false
      if (!this.rangeStart) {
        return true;
      }
      if (event.type === "mouseout") {
        return true;
      }
      while (
        this.$el.contains(target) &&
        !~target.className.indexOf("day-cell")
      ) {
        target = target.parentNode;
      }
      if (
        ~target.className.indexOf("day-cell") &&
        !~target.className.indexOf("datepicker-item-gray")
      ) {
        const rangeEnd = target.getAttribute("data-date");
        if (this.rangeStart < this.parse(rangeEnd)) {
          this.rangeEnd = this.parse(rangeEnd);
        }
      }
    },
    __OnDrawDate(e) {
      let date = e.date;
      let maxDate = this.parse(this.maxDate, false);
      let minDate = this.parse(this.minDate, false);
      if (this.isDate(maxDate)) {
        if (date.getTime() > maxDate.getTime()) {
          e.allowSelect = false;
        }
      }
      if (this.isDate(minDate)) {
        if (date.getTime() < minDate.getTime()) {
          e.allowSelect = false;
        }
      }
      this.$emit("drawdate", e);
      this.onDrawDate(e);
    },
    getItemClasses(d) {
      d.allowSelect = true;
      this.__OnDrawDate(d);
      const clazz = [];
      clazz.push(d.sclass);
      if (
        this.rangeStart &&
        this.rangeEnd &&
        d.sclass !== "datepicker-item-gray"
      ) {
        if (d.date > this.rangeStart && d.date < this.rangeEnd) {
          clazz.push("daytoday-range");
        }
        /* eslint-disable eqeqeq */
        if (this.stringify(d.date) == this.stringify(this.rangeStart)) {
          clazz.push("daytoday-start");
        }
        /* eslint-disable eqeqeq */
        if (this.stringify(d.date) == this.stringify(this.rangeEnd)) {
          clazz.push("daytoday-end");
        }
      }
      if (d.allowSelect == false) {
        clazz.push("datepicker-item-disabled");
      }
      return clazz.join(" ");
    },
    translations(lang) {
      lang = lang || "zh-CN";
      var text = {
        daysOfWeek: ["日", "一", "二", "三", "四", "五", "六"],
        limit: "超过限制 (最多{{limit}}项)",
        loading: "加载中...",
        minLength: "Min. Length",
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
      };
      lang = lang.toLowerCase();
      switch (lang) {
        case "zh-cn":
          text = LANG_ZH_CN;
          break;
        case "zh-tw":
          text = LANG_ZH_TW;
          break;
        case "en":
          text = LANG_EN;
          break;
        default:
          break;
      }
      return window.VueCalendarLang ? window.VueCalendarLang(lang) : text;
      // return text;
    },
    close() {
      this.displayDayView = this.displayMonthView = this.displayYearView = false;
    },
    inputClick() {
      this.currDate = this.parse(this.inputValue) || this.parse(new Date());
      if (this.displayMonthView || this.displayYearView) {
        this.displayDayView = false;
      } else {
        this.displayDayView = !this.displayDayView;
      }
      this.updatePaneStyle();
    },
    getElOffset(el) {
      let top = el.offsetTop;
      let left = el.offsetLeft;
      let offsetParent = el.offsetParent;
      while (offsetParent && offsetParent != document.body) {
        top += offsetParent.offsetTop;
        left += offsetParent.offsetLeft;
        offsetParent = offsetParent.offsetParent;
      }
      return {
        top,
        left
      };
    },
    updatePaneStyle() {
      if (!(this.displayMonthView || this.displayYearView)) {
        this.$nextTick(function() {
          let { left, top } = this.getElOffset(this.$el);
          let offsetLeft = this.$el.offsetLeft;
          let elWidth = this.$el.offsetWidth;
          let offsetTop = top + this.$el.offsetHeight;
          let offsetWidth = this.$refs.popup.querySelector(".datepicker-inner")
            .offsetWidth;
          let popWidth = this.pane * offsetWidth + this.borderWidth; // add border
          this.paneStyle.width = popWidth + "px";
          if (this.hasInput) {
            if (this.transfer) {
              this.paneStyle.left = left + "px";
              this.paneStyle.top = offsetTop + "px";
              if (popWidth + left > document.documentElement.clientWidth) {
                this.paneStyle.left = left + elWidth - popWidth + "px";
              }
            } else {
              if (
                popWidth + offsetLeft >
                document.documentElement.clientWidth
              ) {
                this.paneStyle.right = "0px";
              }
            }
          } else {
            this.paneStyle.position = "initial";
          }
          this.$forceUpdate();
        });
      }
    },
    preNextDecadeClick(flag) {
      const year = this.currDate.getFullYear();
      const months = this.currDate.getMonth();
      const date = this.currDate.getDate();
      if (flag === 0) {
        this.currDate = new Date(year - 10, months, date);
      } else {
        this.currDate = new Date(year + 10, months, date);
      }
    },
    preNextMonthClick(flag) {
      const year = this.currDate.getFullYear();
      const month = this.currDate.getMonth();
      const date = this.currDate.getDate();
      if (flag === 0) {
        const preMonth = this.getYearMonth(year, month - 1);
        const lastDate = Math.min(
          this.getDayCount(preMonth.year, preMonth.month),
          date
        );
        this.currDate = new Date(preMonth.year, preMonth.month, lastDate);
        this.changePane(preMonth.year, preMonth.month, this.pane);
      } else {
        const nextMonth = this.getYearMonth(year, month + 1);
        const lastDate = Math.min(
          this.getDayCount(nextMonth.year, nextMonth.month),
          date
        );
        this.currDate = new Date(nextMonth.year, nextMonth.month, lastDate);
        this.changePane(nextMonth.year, nextMonth.month, this.pane);
      }
    },
    preNextYearClick(flag) {
      const year = this.currDate.getFullYear();
      const months = this.currDate.getMonth();
      const date = this.currDate.getDate();
      if (flag === 0) {
        this.currDate = new Date(year - 1, months, date);
      } else {
        this.currDate = new Date(year + 1, months, date);
      }
    },
    yearSelect(year) {
      this.displayYearView = false;
      this.displayMonthView = true;
      this.currDate = new Date(
        year,
        this.currDate.getMonth(),
        this.currDate.getDate()
      );
    },
    daySelect(item, event) {
      let date = item.date;
      let el = event.target;
      if (
        item.allowSelect == false ||
        el.classList[0] === "datepicker-item-disable"
      ) {
        return false;
      } else {
        if (this.hasInput) {
          this.currDate = date;
          this.inputValue = this.stringify(this.currDate);
          this.displayDayView = false;
        }
        // else {
        this.onDayClick(date, this.stringify(date)); //无论有没有 input 文本框，只要选择了日期都触发事件
        // }
      }
    },
    switchMonthView() {
      if (this.showDateOnly) {
        return true;
      }
      this.displayDayView = false;
      this.displayMonthView = true;
    },
    switchDecadeView() {
      this.displayMonthView = false;
      this.displayYearView = true;
    },
    monthSelect(year, index) {
      this.displayMonthView = false;
      this.displayDayView = true;
      this.currDate = new Date(year, index, this.currDate.getDate());
      this.changePane(year, index, this.pane);
    },
    getYearMonth(year, month) {
      if (month > 11) {
        year++;
        month = 0;
      } else if (month < 0) {
        year--;
        month = 11;
      }
      return { year: year, month: month };
    },
    getSpecailDay(v) {
      return this.specialDays[this.stringify(v)];
    },
    stringifyDecadeHeader(date, pan) {
      const yearStr = date.getFullYear().toString();
      const firstYearOfDecade =
        parseInt(yearStr.substring(0, yearStr.length - 1) + 0, 10) + pan * 10;
      const lastYearOfDecade = parseInt(firstYearOfDecade, 10) + 10;
      return firstYearOfDecade + "-" + lastYearOfDecade;
    },
    siblingsMonth(v, n) {
      return new Date(v.getFullYear(), v.getMonth() * 1 + n);
    },
    stringifyDayHeader(date, month = 0) {
      const d = this.siblingsMonth(date, month);
      return this.text.months[d.getMonth()] + " " + d.getFullYear();
    },
    parseMonth(date) {
      return this.text.months[date.getMonth()];
    },
    stringifyYearHeader(date, year = 0) {
      return date.getFullYear() + year;
    },
    isDate(value) {
      return !!(value && value.getFullYear);
    },
    stringify(date, format = this.dateFormat) {
      if (!date) date = this.parse(this.inputValue);
      if (!date) return "";
      const year = date.getFullYear();
      const month = date.getMonth() + 1;
      const day = date.getDate();
      const monthName = this.parseMonth(date);
      return format
        .replace(/yyyy/g, year)
        .replace(/MMMM/g, monthName)
        .replace(/MMM/g, monthName.substring(0, 3))
        .replace(/MM/g, ("0" + month).slice(-2))
        .replace(/dd/g, ("0" + day).slice(-2))
        .replace(/yy/g, year)
        .replace(/M(?!a)/g, month)
        .replace(/d/g, day);
    },
    parse(str, safe = true) {
      if (typeof str == "string") {
        let date;
        if (
          str.length === 10 &&
          (this.dateFormat === "dd-MM-yyyy" || this.dateFormat === "dd/MM/yyyy")
        ) {
          date = new Date(
            str.substring(6, 10),
            str.substring(3, 5) - 1,
            str.substring(0, 2)
          );
        } else {
          date = new Date(str);
          date.setHours(0, 0, 0);
        }
        return isNaN(date.getFullYear()) ? (safe ? new Date() : date) : date;
      } else return str;
    },
    getDayCount(year, month) {
      const dict = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
      if (month === 1) {
        if (year % 400 === 0 || (year % 4 === 0 && year % 100 !== 0)) {
          return 29;
        }
      }
      return dict[month];
    },
    //
    prefixLen(date) {
      const firstDay = this.firstDayOfWeek;
      const wkday = date.getDay(); // frist Date
      return wkday >= firstDay ? wkday - firstDay : 7 - firstDay + wkday;
    },
    getDateRange() {
      this.dateRange = [];
      this.decadeRange = [];
      for (let p = 0; p < this.pane; p++) {
        let currMonth = this.siblingsMonth(this.currDate, p);
        let time = {
          year: currMonth.getFullYear(),
          month: currMonth.getMonth()
        };
        let yearStr = time.year.toString();
        this.decadeRange[p] = [];
        let firstYearOfDecade =
          yearStr.substring(0, yearStr.length - 1) + 0 - 1;
        for (let i = 0; i < 12; i++) {
          this.decadeRange[p].push({
            text: firstYearOfDecade + i + p * 10
          });
        }
        this.dateRange[p] = [];
        const currMonthFirstDay = new Date(time.year, time.month, 1);
        let firstDayWeek = this.prefixLen(currMonthFirstDay);
        const dayCount = this.getDayCount(time.year, time.month);
        if (firstDayWeek >= 1) {
          const preMonth = this.getYearMonth(time.year, time.month - 1);
          const prevMonthDayCount = this.getDayCount(
            preMonth.year,
            preMonth.month
          );
          for (let i = 0; i < firstDayWeek; i++) {
            const dayText = prevMonthDayCount - firstDayWeek + i + 1;
            this.dateRange[p].push({
              text: dayText,
              date: new Date(preMonth.year, preMonth.month, dayText),
              sclass: "datepicker-item-gray"
            });
          }
        }
        for (let i = 1; i <= dayCount; i++) {
          const date = new Date(time.year, time.month, i);
          const week = date.getDay();
          let sclass = "";
          this.disabledDaysOfWeek.forEach(el => {
            if (week === parseInt(el, 10)) sclass = "datepicker-item-disable";
          });
          if (i === this.currDate.getDate()) {
            if (this.inputValue) {
              const valueDate = this.parse(this.inputValue);
              if (valueDate) {
                if (
                  valueDate.getFullYear() === time.year &&
                  valueDate.getMonth() === time.month
                ) {
                  sclass = "datepicker-dateRange-item-active";
                }
              }
            }
          }
          this.dateRange[p].push({
            text: i,
            date: date,
            sclass: sclass
          });
        }
        if (this.dateRange[p].length < 42) {
          const nextMonthNeed = 42 - this.dateRange[p].length;
          const nextMonth = this.getYearMonth(time.year, time.month + 1);
          for (let i = 1; i <= nextMonthNeed; i++) {
            this.dateRange[p].push({
              text: i,
              date: new Date(nextMonth.year, nextMonth.month, i),
              sclass: "datepicker-item-gray"
            });
          }
        }
      }
    }
  }
};
</script>

<style lang="css">/*!
 * vue2-calendar v2.2.5
 * (c) 2019 Alax <76299199@qq.com>
 * https://github.com/icai/vue2-calendar#readme
 */
.datepicker {
  font-size: 14px;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  box-sizing: border-box;
  -webkit-tap-highlight-color: #0000;
  -webkit-font-smoothing: antialiased;
  color: rgba(0, 0, 0, 0.65098);
  position: relative;
  display: inline-block;
}
.datepicker .form-control {
    box-sizing: border-box;
    display: block;
    width: 100%;
    height: 34px;
    padding: 6px 12px;
    font-size: 14px;
    line-height: 1.42857;
    color: #555;
    background-color: #fff;
    background-image: none;
    border: 1px solid #d9d9d9;
    border-radius: 4px;
    box-shadow: none;
}
.datepicker .form-control:hover, .datepicker .form-control:focus {
      outline: 0;
      border-color: #40a9ff;
      box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075), 0 0 8px rgba(102, 175, 233, 0.6);
}
.datepicker button.close {
    padding: 0;
    cursor: pointer;
    background: #0000;
    border: 0;
    -webkit-appearance: none;
    float: right;
    font-size: 21px;
    font-weight: bold;
    line-height: 1;
    color: #000;
    text-shadow: 0 1px 0 #fff;
    opacity: 0.2;
    filter: alpha(opacity=20);
}
.datepicker button.datepicker-input-icon {
    padding: 0;
    cursor: pointer;
    background: #0000;
    border: 0;
    -webkit-appearance: none;
    float: right;
    font-size: 21px;
    font-weight: bold;
    line-height: 1;
    color: #000;
    text-shadow: 0 1px 0 #fff;
    opacity: 0.2;
    filter: alpha(opacity=20);
}
input.datepicker-input.with-reset-button {
  padding-right: 25px;
}
.datepicker > button.close {
  position: absolute;
  top: 0;
  right: 0;
  outline: none;
  z-index: 2;
  display: block;
  width: 34px;
  height: 34px;
  line-height: 34px;
  text-align: center;
}
.datepicker > button.datepicker-input-icon {
  position: absolute;
  top: 0;
  right: 0;
  outline: none;
  z-index: 2;
  display: block;
  width: 34px;
  height: 34px;
  line-height: 34px;
  text-align: center;
}
.datepicker > button.close:hover {
  opacity: 1;
}
.datepicker > button.datepicker-input-icon:hover,
.datepicker > .datepicker-input:focus + button.datepicker-input-icon,
.datepicker-input:hover + button.datepicker-input-icon {
  opacity: 0.7;
}
.datepicker-wrapper {
  position: absolute;
  z-index: 1000;
}
.datepicker-wrapper.datepicker-wrapper-right {
  right: 0;
}
.datepicker-popup {
  border: 1px solid #fff;
  border-radius: 5px;
  background: #fff;
  margin-top: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}
.datepicker-popup:before, .datepicker-popup:after {
    content: " ";
    display: table;
}
.datepicker-popup:after {
    clear: both;
}
.datepicker-inner {
  width: 218px;
  float: left;
}
.datepicker-body {
  padding: 10px 10px;
  text-align: center;
}
.datepicker-body p {
    margin: 0 0 10px;
}
.datepicker-ctrl p,
.datepicker-ctrl span,
.datepicker-body span {
  display: inline-block;
  width: 28px;
  line-height: 28px;
  height: 28px;
}
.datepicker-ctrl p {
  width: 65%;
}
.datepicker-ctrl span {
  position: absolute;
}
.datepicker-body span {
  text-align: center;
}
.datepicker-monthRange span {
  width: 48px;
  height: 50px;
  line-height: 45px;
}
.datepicker-item-disable {
  background-color: white !important;
  cursor: not-allowed !important;
}
.decadeRange span:first-child,
.decadeRange span:last-child,
.datepicker-item-disable,
.datepicker-item-gray {
  color: #999;
}
.datepicker-dateRange-item-active:hover,
.datepicker-dateRange-item-active {
  background: #3276b1 !important;
  color: white !important;
}
.datepicker-item-disabled {
  color: #aaa;
  text-decoration: line-through;
}
.datepicker-monthRange {
  margin-top: 10px;
}
.datepicker-monthRange span,
.datepicker-ctrl span,
.datepicker-ctrl p,
.datepicker-dateRange span {
  cursor: pointer;
}
.datepicker-monthRange span:hover,
.datepicker-ctrl p:hover,
.datepicker-ctrl i:hover,
.datepicker-dateRange span:hover,
.datepicker-dateRange-item-hover {
  background-color: #eeeeee;
}
.datepicker-dateRange .daytoday-start,
.datepicker-dateRange .daytoday-start:hover,
.datepicker-dateRange .daytoday-end,
.datepicker-dateRange .daytoday-end:hover {
  background: #3276b1 !important;
  color: white !important;
}
.datepicker-dateRange .daytoday-range,
.datepicker-dateRange .daytoday-range:hover {
  background-color: #ddd;
}
.datepicker-weekRange span {
  font-weight: bold;
}
.datepicker-label {
  background-color: #f8f8f8;
  font-weight: 700;
  padding: 7px 0;
  text-align: center;
}
.datepicker-ctrl {
  position: relative;
  /*height: 30px;*/
  line-height: 30px;
  font-weight: bold;
  text-align: center;
  top: 3px;
}
.month-btn {
  font-weight: bold;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
.datepicker-preBtn {
  left: 2px;
  font-size: 18px;
}
.datepicker-nextBtn {
  right: 2px;
  font-size: 18px;
}
@font-face {
  font-family: "calendar";
  src: url(data:application/vnd.ms-fontobject;base64,0AgAACgIAAABAAIAAAAAAAIABQMAAAAAAAABAJABAAAAAExQAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAADy5+aAAAAAAAAAAAAAAAAAAAAAAAABAAaQBjAG8AbgBmAG8AbgB0AAAADgBSAGUAZwB1AGwAYQByAAAAFgBWAGUAcgBzAGkAbwBuACAAMQAuADAAAAAQAGkAYwBvAG4AZgBvAG4AdAAAAAAAAAEAAAALAIAAAwAwR1NVQrD+s+0AAAE4AAAAQk9TLzI8iEhbAAABfAAAAFZjbWFws9gb8QAAAeQAAAGcZ2x5ZjhX+8UAAAOMAAAB6GhlYWQUsc96AAAA4AAAADZoaGVhB94DhQAAALwAAAAkaG10eBAAAAAAAAHUAAAAEGxvY2EBkgDGAAADgAAAAAptYXhwAR0AgAAAARgAAAAgbmFtZT5U/n0AAAV0AAACbXBvc3SzmBu0AAAH5AAAAEEAAQAAA4D/gABcBAAAAAAABAAAAQAAAAAAAAAAAAAAAAAAAAQAAQAAAAEAAGh+Lg9fDzz1AAsEAAAAAADYyUW/AAAAANjJRb8AAP9/BAADgAAAAAgAAgAAAAAAAAABAAAABAB0AA8AAAAAAAIAAAAKAAoAAAD/AAAAAAAAAAEAAAAKAB4ALAABREZMVAAIAAQAAAAAAAAAAQAAAAFsaWdhAAgAAAABAAAAAQAEAAQAAAABAAgAAQAGAAAAAQAAAAAAAQQAAZAABQAIAokCzAAAAI8CiQLMAAAB6wAyAQgAAAIABQMAAAAAAAAAAAAAAAAAAAAAAAAAAAAAUGZFZABA5hHmaQOA/4AAXAOAAIEAAAABAAAAAAAABAAAAAQAAAAEAAAABAAAAAAAAAUAAAADAAAALAAAAAQAAAFoAAEAAAAAAGIAAwABAAAALAADAAoAAAFoAAQANgAAAAgACAACAADmEeYe5mn//wAA5hHmHuZp//8AAAAAAAAAAQAIAAgACAAAAAIAAwABAAABBgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMAAAAAAA0AAAAAAAAAAMAAOYRAADmEQAAAAIAAOYeAADmHgAAAAMAAOZpAADmaQAAAAEAAAAAAJ4AxgD0AAAADwAA/38D3AOAAAMABwALAA8AEwAXABsAHwAjADMANwA7AD8ATwBzAAAXMzUjFzM1IyczNSMXMzUjJzM1IwEzNSMDMzUjATM1IyczNSMDNTQmKwEiBh0BFBY7ATI2ATM1IyczNSMXMzUjNzU0JisBIgYdARQWOwEyNjcRFAYjISImNRE0NjsBNTQ2OwEyFh0BMzU0NjsBMhYdATMyFm6kpMm3t8mkpMm3t8mkpAGkt7fbt7cBt6Sk3Le3yQsHJQcLCwclBwsBpaSk3Le33KSkEgsHJQcLCwclBwvbKx783B4rKx5JNiUlJjXcNSYlJTZJHis3paWlJLe3tyWk/belAQCk/belJLcBN6UHCwsHpQcLC/4ZtyWkpKRupQcLCwelBwsLLP0kHisrHgLcHSw3JTY2JTc3JTY2JTcsAAEAAP/BAu8DQwAUAAAFBiInASY0NwE2MhYUBwEGFBcBFhQC3g4jDv6OGxsBdQ0kGw3+qw0NAVINMQ0NAXIcRxwBdQ0aJQ3+qg4jDv6tDiMAAQAA//8DAgMCABYAAAE2NzYmJwEmDgEWFwkBDgEWMjcBPwE2AvkBAQYEDP6fDyccAQ8BPf7EDgEdJg8BXgECBAFsAQIOHgsBTg4BHiYP/tX+zw4mHg0BUgIBBQAAABIA3gABAAAAAAAAABUAAAABAAAAAAABAAgAFQABAAAAAAACAAcAHQABAAAAAAADAAgAJAABAAAAAAAEAAgALAABAAAAAAAFAAsANAABAAAAAAAGAAgAPwABAAAAAAAKACsARwABAAAAAAALABMAcgADAAEECQAAACoAhQADAAEECQABABAArwADAAEECQACAA4AvwADAAEECQADABAAzQADAAEECQAEABAA3QADAAEECQAFABYA7QADAAEECQAGABABAwADAAEECQAKAFYBEwADAAEECQALACYBaQpDcmVhdGVkIGJ5IGljb25mb250Cmljb25mb250UmVndWxhcmljb25mb250aWNvbmZvbnRWZXJzaW9uIDEuMGljb25mb250R2VuZXJhdGVkIGJ5IHN2ZzJ0dGYgZnJvbSBGb250ZWxsbyBwcm9qZWN0Lmh0dHA6Ly9mb250ZWxsby5jb20ACgBDAHIAZQBhAHQAZQBkACAAYgB5ACAAaQBjAG8AbgBmAG8AbgB0AAoAaQBjAG8AbgBmAG8AbgB0AFIAZQBnAHUAbABhAHIAaQBjAG8AbgBmAG8AbgB0AGkAYwBvAG4AZgBvAG4AdABWAGUAcgBzAGkAbwBuACAAMQAuADAAaQBjAG8AbgBmAG8AbgB0AEcAZQBuAGUAcgBhAHQAZQBkACAAYgB5ACAAcwB2AGcAMgB0AHQAZgAgAGYAcgBvAG0AIABGAG8AbgB0AGUAbABsAG8AIABwAHIAbwBqAGUAYwB0AC4AaAB0AHQAcAA6AC8ALwBmAG8AbgB0AGUAbABsAG8ALgBjAG8AbQAAAAACAAAAAAAAAAoAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQBAgEDAQQBBQAIY2FsZW5kYXIETGVmdAVyaWdodAAAAAAA);
  src: url(data:application/vnd.ms-fontobject;base64,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#iefix) format("embedded-opentype"), url(data:font/ttf;base64,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) format("truetype"), url(data:font/woff2;base64,d09GMgABAAAAAAOkAAsAAAAACCgAAANXAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDHAqDaIM1ATYCJAMQCwoABCAFhG0HQRsgBxHVm4XIvjiw6ciJva5pWPivQy6KSUy39WMQoc6aleREJx/DM1ONyC2T8yVwRZ0DIG7zL/h/cc4DGxV1OZKzB1ToC0k7oCyn/zlmvLIEzL9KsCllxPPR3nC0N94u9MAqSvgaz/2W4bUuhYM+GgI4CqmEtGrTqRcWA1lJAJk0fuxwbMaJaegULEKkYkuDrMXDqmXqIbAm+Hj5Tg2xoPA0ZM1uY1qPotmnlE/j9dxwLoM8N/Om8wPydqCBSoABGVcZHIGOipXQOHN+Qx3AYlEw1azCT+PDsfooRktDoAb/8yQCaY8HoBaUNHxK4Vwo+FSorDKNB/UDHMkAYIEUYC9wE4LNhy8JQYwhGbOXysjq7Yvr7RNIG//7ATJvs3PLN8LcTb6/Ysvorv36Vq/f18y96ZKX3xZv3h3fl3kvjOFnEL3hiluTNy7PHHN8XsWxW1Ycm5s+auPcjNGb5s7ckrVle+aYzReP7soYPWrjzmMrssZmZW2ZXOfgoYN3Dx66crekJwPi5Su2UgeJ/UC7Yv378OCVy+3qHR9KltbVORQ9Hi3N1YH94Kfu9g+lWtg/HIBBKZOb6t9XDAqPF5biawUFJwrb1y3du3TpMq1L1w0if+sypTNnq9u+8ASxakHJYPr0E21PzJr1sNSx0g979X5U8lH3Hu498n+iDG58fVjnn8Nueb8aVWhYLeY9KvNjcHdJrsU0AEI60HMDcuq3bulbK6r+vX9Dv9mFU4cmNfpjfQPA23ut7+BEyDBM9Y9UnVU/U9Wxx1ga4jLG0HKlfSLDLJIAUIDDAb+wCql8nCKcAYlfsIoKQBGlADSWUgiDqQQePrUggqUpOCrSdnufNGagERMHVGARAiGZM6BI5A5oknmEMJgYeGTyEyIkiwZHH0nbo4+/lsxtZqAStKD/kJmmJuaRVdh5Q/cuFac1AemF3GcZiIMoXzlgjTzFOv3HJSIExE0Fe3Q8LMsGWm5yNBJ4kTYNQ6p6pMA01WDFhlEJWkD/gcw0NXmjZVV4/xu6d6m4oasp+ULus7kDMQFRB8TBWHfqupXr9B+XIEIAcVMBe+THkhYboK1eLEcjAX5Avk0JcT/qagyWN1dveAxwkAsbUaLFiMdi5PASa6t4ch1NMuXxzstgAAA=) format("woff2"), url(data:font/woff;base64,d09GRgABAAAAAAVMAAsAAAAACCgAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAABHU1VCAAABCAAAADMAAABCsP6z7U9TLzIAAAE8AAAARAAAAFY8iEhbY21hcAAAAYAAAABfAAABnLPYG/FnbHlmAAAB4AAAAWkAAAHoOFf7xWhlYWQAAANMAAAALwAAADYUsc96aGhlYQAAA3wAAAAcAAAAJAfeA4VobXR4AAADmAAAAA4AAAAQEAAAAGxvY2EAAAOoAAAACgAAAAoBkgDGbWF4cAAAA7QAAAAfAAAAIAEdAIBuYW1lAAAD1AAAAUUAAAJtPlT+fXBvc3QAAAUcAAAAMAAAAEGzmBu0eJxjYGRgYOBikGPQYWB0cfMJYeBgYGGAAJAMY05meiJQDMoDyrGAaQ4gZoOIAgCKIwNPAHicY2BkYWCcwMDKwMHUyXSGgYGhH0IzvmYwYuRgYGBiYGVmwAoC0lxTGByeCT7LZG7438AQw9zA0AgUZgTJAQDmXgxeeJztkLENgDAMBM9JQBGiYYFMQIkYiIosnjWCY1MwBG+dZb8sFw9MQFR2JYHcCEOXumJ+ZDE/ceqetQK0rZVWe/9OJrGLrFPwDzLza7V+vFscqTmWZXHMrw7yAOCDE4oAeJxdkD1Ow0AQheetwREKtgVYjiCJEXF2UzgNJt4tIkRBgWhA4gCcAFGAqOEEXIFm3YCot0aioaKiwuIK1EhgWIdfUcw3O29m3kpDAdH7qVM6Z+RQg5oU0AK1qE3LlNAaKRrTBu3SMVFrLU/qGPzJsOF85bp28pFI0XO7CKMxMok/8+pfT82FbrLSE/ncSI6R18iirt34eWXRodZ3xtx9E9qYR2NgtC6t0GzwRnMCFBOl1Hr+V3xM49cyTtN4W3Iu8jIXnMvtOFVFUfSNMVy/mQJUs2+ginqtRrVoW1of/gjDt35tw8ruUHEpufrkkGBvd8OenU0Kiabd3gBipCCzKGzADVuIQvbkJ3513m7jxOu3verK87DnrVoedbY6VlziXnVZz1z7ycTv3WEOo4gIUklhHX1ErRlYZgobkOwFcKdmq4tg0EGA9erWR1cE2AebwgGYHzex4yMWQfVQ3fsith8yTH8ACsRw/gAAAHicY2BkYGAA4ow6ufx4fpuvDNwsDCBw46TrfgT9v56FgbkByOVgYAKJAgAkCAp9AHicY2BkYGBu+N/AEMPCAAJAkpEBFbAAAEcKAm14nGNhYGBgQcIAALAAEQAAAAAAAACeAMYA9AAAeJxjYGRgYGBhKGHgZwABJiDmAkIGhv9gPgMAFWMBngB4nGWPTU7DMBCFX/oHpBKqqGCH5AViASj9EatuWFRq911036ZOmyqJI8et1ANwHo7ACTgC3IA78EgnmzaWx9+8eWNPANzgBx6O3y33kT1cMjtyDRe4F65TfxBukF+Em2jjVbhF/U3YxzOmwm10YXmD17hi9oR3YQ8dfAjXcI1P4Tr1L+EG+Vu4iTv8CrfQ8erCPuZeV7iNRy/2x1YvnF6p5UHFockikzm/gple75KFrdLqnGtbxCZTg6BfSVOdaVvdU+zXQ+ciFVmTqgmrOkmMyq3Z6tAFG+fyUa8XiR6EJuVYY/62xgKOcQWFJQ6MMUIYZIjK6Og7VWb0r7FDwl57Vj3N53RbFNT/c4UBAvTPXFO6stJ5Ok+BPV8bUnV0K27LnpQ0kV7NSRKyQl7WtlRC6gE2ZVeOEXpc0Yk/KGdI/wAJWm7IAAAAeJxjYGKAAC4G7ICFkYmRmZGFkZWBIzkxJzUvJbGIxSc1rYS1KDM9o4SBAQBZMQcX) format("woff"), url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBzdGFuZGFsb25lPSJubyI/Pgo8IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiID4KPCEtLQoyMDEzLTktMzA6IENyZWF0ZWQuCi0tPgo8c3ZnPgo8bWV0YWRhdGE+CkNyZWF0ZWQgYnkgaWNvbmZvbnQKPC9tZXRhZGF0YT4KPGRlZnM+Cgo8Zm9udCBpZD0iaWNvbmZvbnQiIGhvcml6LWFkdi14PSIxMDI0IiA+CiAgPGZvbnQtZmFjZQogICAgZm9udC1mYW1pbHk9Imljb25mb250IgogICAgZm9udC13ZWlnaHQ9IjUwMCIKICAgIGZvbnQtc3RyZXRjaD0ibm9ybWFsIgogICAgdW5pdHMtcGVyLWVtPSIxMDI0IgogICAgYXNjZW50PSI4OTYiCiAgICBkZXNjZW50PSItMTI4IgogIC8+CiAgICA8bWlzc2luZy1nbHlwaCAvPgogICAgCiAgICA8Z2x5cGggZ2x5cGgtbmFtZT0iY2FsZW5kYXIiIHVuaWNvZGU9IiYjNTg5ODU7IiBkPSJNMTA5LjcxNDI4Ni01NC44NTcxNDNsMTY0LjU3MTQyOSAwIDAgMTY0LjU3MTQyOS0xNjQuNTcxNDI5IDAgMC0xNjQuNTcxNDI5em0yMDEuMTQyODU3IDBsMTgyLjg1NzE0MyAwIDAgMTY0LjU3MTQyOS0xODIuODU3MTQzIDAgMC0xNjQuNTcxNDI5em0tMjAxLjE0Mjg1NyAyMDEuMTQyODU3bDE2NC41NzE0MjkgMCAwIDE4Mi44NTcxNDMtMTY0LjU3MTQyOSAwIDAtMTgyLjg1NzE0M3ptMjAxLjE0Mjg1NyAwbDE4Mi44NTcxNDMgMCAwIDE4Mi44NTcxNDMtMTgyLjg1NzE0MyAwIDAtMTgyLjg1NzE0M3ptLTIwMS4xNDI4NTcgMjE5LjQyODU3MWwxNjQuNTcxNDI5IDAgMCAxNjQuNTcxNDI5LTE2NC41NzE0MjkgMCAwLTE2NC41NzE0Mjl6bTQyMC41NzE0MjktNDIwLjU3MTQyOWwxODIuODU3MTQzIDAgMCAxNjQuNTcxNDI5LTE4Mi44NTcxNDMgMCAwLTE2NC41NzE0Mjl6bS0yMTkuNDI4NTcxIDQyMC41NzE0MjlsMTgyLjg1NzE0MyAwIDAgMTY0LjU3MTQyOS0xODIuODU3MTQzIDAgMC0xNjQuNTcxNDI5em00MzguODU3MTQzLTQyMC41NzE0MjlsMTY0LjU3MTQyOSAwIDAgMTY0LjU3MTQyOS0xNjQuNTcxNDI5IDAgMC0xNjQuNTcxNDI5em0tMjE5LjQyODU3MSAyMDEuMTQyODU3bDE4Mi44NTcxNDMgMCAwIDE4Mi44NTcxNDMtMTgyLjg1NzE0MyAwIDAtMTgyLjg1NzE0M3ptLTIwMS4xNDI4NTcgNDkzLjcxNDI4NmwwIDE2NC41NzE0MjlxMCA3LjQyODU3MS01LjQyODU3MSAxMi44NTcxNDN0LTEyLjg1NzE0MyA1LjQyODU3MWwtMzYuNTcxNDI5IDBxLTcuNDI4NTcxIDAtMTIuODU3MTQzLTUuNDI4NTcxdC01LjQyODU3MS0xMi44NTcxNDNsMC0xNjQuNTcxNDI5cTAtNy40Mjg1NzEgNS40Mjg1NzEtMTIuODU3MTQzdDEyLjg1NzE0My01LjQyODU3MWwzNi41NzE0MjkgMHE3LjQyODU3MSAwIDEyLjg1NzE0MyA1LjQyODU3MXQ1LjQyODU3MSAxMi44NTcxNDN6bTQyMC41NzE0MjktNDkzLjcxNDI4NmwxNjQuNTcxNDI5IDAgMCAxODIuODU3MTQzLTE2NC41NzE0MjkgMCAwLTE4Mi44NTcxNDN6bS0yMTkuNDI4NTcxIDIxOS40Mjg1NzFsMTgyLjg1NzE0MyAwIDAgMTY0LjU3MTQyOS0xODIuODU3MTQzIDAgMC0xNjQuNTcxNDI5em0yMTkuNDI4NTcxIDBsMTY0LjU3MTQyOSAwIDAgMTY0LjU3MTQyOS0xNjQuNTcxNDI5IDAgMC0xNjQuNTcxNDI5em0xOC4yODU3MTQgMjc0LjI4NTcxNGwwIDE2NC41NzE0MjlxMCA3LjQyODU3MS01LjQyODU3MSAxMi44NTcxNDN0LTEyLjg1NzE0MyA1LjQyODU3MWwtMzYuNTcxNDI5IDBxLTcuNDI4NTcxIDAtMTIuODU3MTQzLTUuNDI4NTcxdC01LjQyODU3MS0xMi44NTcxNDNsMC0xNjQuNTcxNDI5cTAtNy40Mjg1NzEgNS40Mjg1NzEtMTIuODU3MTQzdDEyLjg1NzE0My01LjQyODU3MWwzNi41NzE0MjkgMHE3LjQyODU3MSAwIDEyLjg1NzE0MyA1LjQyODU3MXQ1LjQyODU3MSAxMi44NTcxNDN6bTIxOS40Mjg1NzEgMzYuNTcxNDI5bDAtNzMxLjQyODU3MXEwLTI5LjcxNDI4Ni0yMS43MTQyODYtNTEuNDI4NTcxdC01MS40Mjg1NzEtMjEuNzE0Mjg2bC04MDQuNTcxNDI5IDBxLTI5LjcxNDI4NiAwLTUxLjQyODU3MSAyMS43MTQyODZ0LTIxLjcxNDI4NiA1MS40Mjg1NzFsMCA3MzEuNDI4NTcxcTAgMjkuNzE0Mjg2IDIxLjcxNDI4NiA1MS40Mjg1NzF0NTEuNDI4NTcxIDIxLjcxNDI4Nmw3My4xNDI4NTcgMCAwIDU0Ljg1NzE0M3EwIDM3LjcxNDI4NiAyNi44NTcxNDMgNjQuNTcxNDI5dDY0LjU3MTQyOSAyNi44NTcxNDNsMzYuNTcxNDI5IDBxMzcuNzE0Mjg2IDAgNjQuNTcxNDI5LTI2Ljg1NzE0M3QyNi44NTcxNDMtNjQuNTcxNDI5bDAtNTQuODU3MTQzIDIxOS40Mjg1NzEgMCAwIDU0Ljg1NzE0M3EwIDM3LjcxNDI4NiAyNi44NTcxNDMgNjQuNTcxNDI5dDY0LjU3MTQyOSAyNi44NTcxNDNsMzYuNTcxNDI5IDBxMzcuNzE0Mjg2IDAgNjQuNTcxNDI5LTI2Ljg1NzE0M3QyNi44NTcxNDMtNjQuNTcxNDI5bDAtNTQuODU3MTQzIDczLjE0Mjg1NyAwcTI5LjcxNDI4NiAwIDUxLjQyODU3MS0yMS43MTQyODZ0MjEuNzE0Mjg2LTUxLjQyODU3MXoiICBob3Jpei1hZHYteD0iMTAyNCIgLz4KCiAgICAKICAgIDxnbHlwaCBnbHlwaC1uYW1lPSJMZWZ0IiB1bmljb2RlPSImIzU4ODk3OyIgZD0iTTczNC4yLTQ5Yy0xNy41LTE3LjUtNDUuOC0xNy41LTYzLjMgMEwzMDAuNyAzMjEuMmMtMzUgMzUtMzUgOTEuNyAwIDEyNi42bDM3Mi45IDM3M2MxNy4zIDE3LjMgNDUuMyAxNy41IDYyLjggMC41IDE3LjktMTcuNCAxOC4xLTQ2LjEgMC41LTYzLjhMMzk1LjcgNDE2LjJjLTE3LjUtMTcuNS0xNy41LTQ1LjggMC02My4zbDMzOC42LTMzOC42YzE3LjQtMTcuNSAxNy40LTQ1LjgtMC4xLTYzLjN6IiAgaG9yaXotYWR2LXg9IjEwMjQiIC8+CgogICAgCiAgICA8Z2x5cGggZ2x5cGgtbmFtZT0icmlnaHQiIHVuaWNvZGU9IiYjNTg5MTA7IiBkPSJNNzYxLjA1NiAzNjMuODcyYzAuNTEyIDAuOTkyIDEuMzQ0IDEuODI0IDEuNzkyIDIuODQ4IDguOCAxOC4zMDQgNS45MiA0MC43MDQtOS42NjQgNTUuNDI0TDM5OS45MzYgNzU2LjI1NmMtMTkuMjY0IDE4LjIwOC00OS42MzIgMTcuMzQ0LTY3Ljg3Mi0xLjg4OC0xOC4yMDgtMTkuMjY0LTE3LjM3Ni00OS42MzIgMS44ODgtNjcuODcybDMxNi45Ni0yOTkuODRMMzM1LjIgODIuMzY4Yy0xOS4wNzItMTguNC0xOS42NDgtNDguNzY4LTEuMjQ4LTY3Ljg3MiA5LjQwOC05Ljc5MiAyMS45ODQtMTQuNjg4IDM0LjU2LTE0LjY4OCAxMiAwIDI0IDQuNDggMzMuMzEyIDEzLjQ0bDM1MC4wNDggMzM3LjM3NmMwLjY3MiAwLjY3MiAwLjkyOCAxLjYgMS42IDIuMzA0IDAuNTEyIDAuNDggMS4wNTYgMC44MzIgMS41NjggMS4zNDQgMi43MiAyLjg0OCA0LjE2IDYuMzM2IDYuMDE2IDkuNnoiICBob3Jpei1hZHYteD0iMTAyNCIgLz4KCiAgICAKCgogIDwvZm9udD4KPC9kZWZzPjwvc3ZnPgo=#calendar) format("svg");
  font-weight: normal;
  font-style: normal;
}
[class^="calendaricon-"],
[class*=" calendaricon-"] {
  /* use !important to prevent issues with browser extensions that change fonts */
  font-family: "calendar" !important;
  /**speak: none;**/
  font-style: normal;
  font-weight: normal;
  font-variant: normal;
  text-transform: none;
  line-height: 1;
  /* Better Font Rendering =========== */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.calendaricon-angle-left:before {
  content: "\E611";
}
.calendaricon-angle-right:before {
  content: "\E61E";
}
.calendaricon-calendar:before {
  content: "\E669";
}

</style>
