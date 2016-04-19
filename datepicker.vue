<template>
  <div @click.stop="" v-show="isPicker" :style="{'left':x+'px','top':y+'px'}" class="datepicker" transition="datepicker" transition-mode="out-in">
    <div class="datepicker-opts center">
      <a href="javascript:;" @click.stop="refresh(-1)" class="arrow fl"></a>
      <a href="javascript:;" @click.stop="refresh(1)" class="arrow fr"></a>
      <select v-model="year" @change="render(year,month)" class="datepicker-select">
        <option v-for="y in years" value="{{ y }}" class="center">{{ y }}</option>
      </select>
      <select v-model="month" @change="render(year,month)"  class="datepicker-select">
        <option v-for="m in months" value="{{ m }}" class="center">{{ m }}</option>
      </select>
    </div>
    <div class="datepicker-weeks">
      <span v-for="week in weeks" class="week" href="javascript:;">{{ week }}</span>
    </div>
    <div class="datepicker-days">
      <a v-for="day in days" href="javascript:;"
      :class="{'today':day.today, 'disabled':day.disabled, 'del':day.del}"
      data-theday="{{ day.theday }}" title="{{ day.title }}"
      @click="select(day, $event)">{{ day.day }}</a>
    </div>
    <p v-if="pickerror.show" class="center tips">{{ pickerror.info }}</p>
  </div>
</template>

<script>
  export default {
    props: {
      isPicker: { type: Boolean, default: false },
      x: { type: Number, default: 0 },
      y: { type: Number, default: 0 },
      theday: { type: String, default: '' },
      begin: { type: String, default: '1970-01-01' },
      end: { type: String, default: '2030-12-31' },
      disables: { type: Array, default: () => [] },
      sep: { type: String, default: '-' }
    },
    data () {
      return {
        years: [],
        months: ['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12'],
        weeks: ['日', '一', '二', '三', '四', '五', '六'],
        days: [],
        year: 0,
        month: 0,
        day: 0,
        today: '',
        pickerror: {
          info: '',
          show: false
        }
      }
    },
    created: function () {
      var self = this
      var now = new Date()
      var yearBegin = +self.begin.split(self.sep)[0]
      var yearEnd = +self.end.split(self.sep)[0]
      self.year = now.getFullYear()
      self.month = now.getMonth() + 1
      self.day = now.getDate()
      self.today = self.year + self.sep + ('0' + self.month).substr(-2) + self.sep + ('0' + self.day).substr(-2)
      for (var i = yearBegin; i <= yearEnd; i++) {
        self.years.push(i)
      }
      self.render(self.year, self.month)
    },
    methods: {
      render: function (y, m) {
        var self = this
        var dayNode = {}
        // 该月第一天是周几，也是上月需要输出的长度
        var firstDayOfMonth = new Date(y, m - 1, 1).getDay()
        // 该月最后一天是几号，也是该月需要输出的长度
        var lastDateOfMonth = new Date(y, m, 0).getDate()
        // 上月最后一天是几号
        var lastDateOfLastMonth = new Date(y, m - 1, 0).getDate()
        // 该月最后一天是周几，关乎下月输出几天
        var lastDayOfMonth = new Date(y, m, 0).getDay()
        var prev = lastDateOfLastMonth - firstDayOfMonth + 1
        self.pickerror.show = false
        self.days = []
        // 上个月
        for (; prev <= lastDateOfLastMonth; prev++) {
          dayNode = {
            day: prev,
            theday: self.rexday(y, m - 1, prev),
            today: false,
            disabled: true,
            del: false,
            title: ''
          }
          self.days.push(dayNode)
        }
        // 该月
        for (var current = 1; current <= lastDateOfMonth; current++) {
          dayNode = {
            day: current,
            theday: self.rexday(y, m, current),
            today: false,
            disabled: false,
            del: false,
            title: ''
          }
          if (dayNode.theday === self.today) {
            dayNode.today = true
          }
          if ((self.calday(self.begin, dayNode.theday) > 0) || (self.calday(dayNode.theday, self.end) > 0)) {
            dayNode.disabled = true
            dayNode.title = '超出范围的无效日期'
          }
          if (self.disables.indexOf(dayNode.theday) !== -1) {
            dayNode.del = true
            dayNode.disabled = true
            dayNode.title = '当天不可选择'
          }
          self.days.push(dayNode)
        }
        // 下个月
        for (var next = 1; next + lastDayOfMonth <= 6; next++) {
          dayNode = {
            day: next,
            theday: self.rexday(y, m + 1, next),
            today: false,
            disabled: true,
            del: false
          }
          self.days.push(dayNode)
        }
      },
      select: function (day, event) {
        var self = this
        if (!self.hasClass(event.target, 'disabled')) {
          self.theday = day.theday
          self.isPicker = false
        } else {
          return false
        }
      },
      refresh: function (vector) {
        var self = this
        var temp = parseInt(self.month, 10) + 11 + vector
        var yearTemp = parseInt(self.year, 10) + Math.floor(temp / 12) - 1
        if (yearTemp >= self.begin.split(self.sep)[0] && yearTemp <= self.end.split(self.sep)[0]) {
          self.month = temp % 12 + 1
          self.year = yearTemp
          self.render(self.year, self.month)
        } else {
          self.pickerror.info = '超出日期选择范围咯'
          self.pickerror.show = true
        }
      },
      rexday: function (y, m, d) {
        var self = this
        return y + self.sep + ('0' + m).substr(-2) + self.sep + ('0' + d).substr(-2)
      },
      calday: function (d1, d2) {
        var self = this
        var d1arr = d1.split(self.sep)
        var d2arr = d2.split(self.sep)
        for (var i = 0; i < 3; i++) {
          if (d1arr[i] !== d2arr[i]) {
            return d1arr[i] - d2arr[i]
          }
        }
      },
      hasClass: function (el, cls) {
        if ('classList' in el && typeof el.classList.contains === 'function') {
          return el.classList.contains(cls)
        } else {
          var classes = el.className.split(/\s+/)
          var i = classes.length
          while (i) {
            if (cls === classes[i]) return true
            i--
          }
          return false
        }
      }
    }
  }
</script>

<style>
  .datepicker, .datepicker * {
    box-sizing: border-box;
    margin: 0; padding: 0;
    font-size: 14px;
  }
  .datepicker{
    z-index: 100;
    position: absolute;
    width: 246px;
    padding: 10px;
    color: #5d6371;
    background: #fff;
    border: 1px solid #dedede;
    border-radius: 2px;
    box-shadow: 1px 1px 3px rgba(0, 0, 0, .16);
    opacity:.95;
    transition: all .5s ease;
  }
  .datepicker-enter, .datepicker-leave {
    opacity: 0;
    transform: translate3d(0,-10px, 0);
  }
  .datepicker:before, .datepicker:after {
    position: absolute;
    left: 30px; top: -10px;
    content: "";
    border:5px solid rgba(0, 0, 0, 0);
    border-bottom-color: #dedede;
  }
  .datepicker:after {
    top: -9px;
    border-bottom-color: #fff;
  }
  .datepicker a {
    color: #5d6371;
    text-decoration: none;
  }
  .datepicker .fl {
    float: left;
  }
  .datepicker .fr {
    float: right;
  }
  .datepicker .center {
    text-align: center;
  }
  .datepicker .tips {
    margin: 6px 0 0 0;
  }
  .datepicker-opts {
    height: 28px;
  }
  .datepicker-opts .arrow {
    display: block;
    width: 32px; height: 24px;
    background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABkAAAAPCAMAAAAmuJTXAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAxlBMVEUAAAAyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzkyNzn///9PrPDQAAAAQHRSTlMAAANv788nBGvuWQZz88kjB3r1xSwJgfe/DYj5uZn6/LIf9ug2l/20IAyF+LwlCH/CKXj0xy4FcPLLae1s7Mwm88TcCwAAAAFiS0dEQYnebE4AAADiSURBVBjTVZBVlsJQEAVfB0jwBJfgbsFda/+rIgbD9NftU61XKTdEi0RjuqeUHotGNFFhiBFPQNLXSUjEjRBJKp2BrOknZhYy6ZSPxMrloVAM6qRYgHzOcjMplStQ1SUkolehUi6JqtVtaJjy2SpiNsCu11SzRbvT/QIPdTttWk3Vg/7gB3ho0IeeGtqMxpN/PZPxCHuoprM5LJyfPc4C5rOpkuXKHbjefG/brN1Rq6V3trXdwf4QksMedlsrePV4OsPF8YlzgfPp+LHHuN5+fLtdjT9Ltfvj+fLl6/m4B16/AdLGHTd6ZBjpAAAAJXRFWHRkYXRlOmNyZWF0ZQAyMDE1LTA3LTMwVDAwOjI1OjM4KzA4OjAw7sVrLgAAACV0RVh0ZGF0ZTptb2RpZnkAMjAxNS0wNy0zMFQwMDoyNTozOCswODowMJ+Y05IAAABOdEVYdHNvZnR3YXJlAEltYWdlTWFnaWNrIDYuOC44LTEwIFExNiB4ODZfNjQgMjAxNS0wNy0xOSBodHRwOi8vd3d3LmltYWdlbWFnaWNrLm9yZwUMnDUAAAAYdEVYdFRodW1iOjpEb2N1bWVudDo6UGFnZXMAMaf/uy8AAAAYdEVYdFRodW1iOjpJbWFnZTo6SGVpZ2h0ADY2OCJfkgEAAAAYdEVYdFRodW1iOjpJbWFnZTo6V2lkdGgAMTA2OJ+1jYsAAAAZdEVYdFRodW1iOjpNaW1ldHlwZQBpbWFnZS9wbmc/slZOAAAAF3RFWHRUaHVtYjo6TVRpbWUAMTQzODE4NzEzONNWD3oAAAARdEVYdFRodW1iOjpTaXplADE1S0JC7LegNwAAAFp0RVh0VGh1bWI6OlVSSQBmaWxlOi8vL2hvbWUvd3d3cm9vdC93d3cuZWFzeWljb24ubmV0L2Nkbi1pbWcuZWFzeWljb24uY24vc3JjLzExOTA1LzExOTA1NzcucG5ne+ipCwAAAABJRU5ErkJggg==');
    background-repeat: no-repeat;
  }
  .datepicker-opts .arrow.fl {
    background-position: 18px 5px;
  }
  .datepicker-opts .arrow.fr {
    background-position: -12px 5px;
  }
  .datepicker-select {
    height: 24px;
    padding: 2px 8px;
    border: 1px solid #dedede;
    border-radius: 3px;
    background: #fff;
    line-height: 24px;
    outline: none;
  }
  .datepicker-weeks span,
  .datepicker-days a {
    display: inline-block;
    width: 32px; height: 32px;
    line-height: 32px;
    vertical-align: middle;
    text-align: center;
  }
  .datepicker-weeks .week {
    font-weight: bold;
  }
  .datepicker-days a:not(.disabled):hover {
    background: #f5f5f5;
    transition: all .3s;
  }
  .datepicker-days .disabled {
    color: #ccc;
    cursor: not-allowed;
  }
  .datepicker-days .del {
    text-decoration: line-through!important;
  }
  .datepicker-days .today {
    color: #ea6153;
  }
</style>
