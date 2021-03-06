<template>
  <div>
    <div ref="chart" style="height:300px"></div>
  </div>
</template>

<script>
  import Highcharts from 'highcharts/highstock';

  export default {
    name: "TimeChart",
    props: {
      data: {
        type: Array,
        required: true
      },
      rssiLog: {
        type: Array,
        required: true
      },
    },

    updateInterval: null,

    mounted() {
      const $vm = this;
      const setTimeFilterDebounced = $vm.$debounce((min, max) => {
        $vm.$root.timefilter.start = Math.floor(min / 1000);
        $vm.$root.timefilter.stop = Math.ceil(max / 1000);
      }, 500);

      this.hightchart = Highcharts.stockChart(this.$refs.chart, {
        chart: {
          zoomType: 'x',
        },
        tooltip: {
          valueDecimals: 0
        },
        xAxis: {
          events: {
            afterSetExtremes(ev) {
              setTimeFilterDebounced(ev.min, ev.max);
            }
          },
          type: 'datetime',
          ordinal: false,
        },
        time: { useUTC: false },
        rangeSelector: {
          inputEnabled: false,
          buttons: [{
            type: 'minute',
            count: 1,
            text: '1m'
          }, {
            type: 'minute',
            count: 5,
            text: '5m'
          }, {
            type: 'minute',
            count: 10,
            text: '10m'
          }, {
            type: 'minute',
            count: 30,
            text: '30m'
          }, {
            type: 'hour',
            count: 1,
            text: '1h'
          }, {
            type: 'all',
            text: 'All'
          }]
        },
        title: { text: 'Telegramme' },
        exporting: { enabled: false },
        yAxis: [
          {
            opposite: false
          },
          {
            opposite: true,
            max: -40,
            min: -120,
          }
        ],
        plotOptions: {
          column: {
            pointPlacement: 'on',
            groupPadding: 0.1,
            dataGrouping: {
              groupPixelWidth: 30
            }
          }
        },
        series: [
          {
            name: 'Telegramme',
            type: 'column',
            maxPointWidth: 15,
          },
          {
            name: 'RSSI Noise',
            type: 'line',
            color: '#ff5200',
            lineWidth: 1,
            yAxis: 1
          }
        ]
      });
      this.updateData();
      this.updateInterval = setInterval(this.updateData.bind(this), 1000);
    },

    beforeDestroy() {
      clearTimeout(this.updateInterval);
    },

    data() {
      return {};
    },

    methods: {
      updateData() {
        if (!this.data.length) return;

        let m = new Map();
        this.data.forEach(t => {
          let cnt = m.get(t.tstamp) || 0;
          cnt++;
          m.set(t.tstamp, cnt);
        });
        let data = [];
        m.forEach((v, k) => data.push([k * 1000, v]));
        data = data.sort((a, b) => a[0] - b[0]);
        this.hightchart.series[0].setData(data, false);
        this.hightchart.series[1].setData(this.rssiLog, false);
        this.hightchart.redraw();
      },
    }
  }
</script>

<style lang="stylus" scoped>

</style>
