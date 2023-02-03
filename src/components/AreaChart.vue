<template>
  <svg :width='chartWidth' :height='chartHeight'>
    <g :transform="`translate(${margin.left},${margin.top})`">
        <path :d="areaPath(data.filter((d) =>  d.country === 'EU27' && d.date <= '2022-03'))" fill="#bdbdbd"/>
        <path :d="areaPath(data.filter((d) =>  d.country === 'EU27' && d.date >= '2022-03'))" fill="#e34a33"/>
        <g class="x-axis" ref='xAxis' :transform='`translate(0,${chartHeight - margin.bottom})`' />
        <g class="y-axis" ref='yAxis' />
    </g>
  </svg>
</template>

  <script>
  import * as d3 from "d3";
  import data from '../assets/data.json'

  export default {
    name: "AreaChart",
    data: () => ({
        scaleFactor: 1.8,
    }),
    computed: {
      data(){
        return data;
      },
      chartWidth() {
        return 1920/this.scaleFactor;
      },
      chartHeight() {
        return 1080/this.scaleFactor;
      },
      margin() {
        return {
          top:  84 / this.scaleFactor,
          left: (70 + 84) / this.scaleFactor,
          bottom: (85 + 84) / this.scaleFactor,
          right:  84 / this.scaleFactor,
        }
      },
      xScale(){
          const minDate = new Date(data[0].date);
          const maxDate = new Date(data[data.length - 1].date);
          return d3.scaleTime()
            .domain([minDate, maxDate])
            .range([0,this.chartWidth - this.margin.left - this.margin.right]);
      },
      yScale(){
          return  d3.scaleLinear()
          .domain([0, d3.max(data, (d) => d.value)])
          .range([this.chartHeight - this.margin.bottom, 0]);
      },
      dateFormat() {
        return d3.timeFormat("%Y %b");
      },
    },
    methods: {
        areaPath(d) {
          const generator = d3.area()
                  .x((d) => this.xScale(new Date(d.date)))
                  .y1((d) => this.yScale(d.value))
                  .y0(this.yScale(0));
          return generator(d)
        },
        renderAxes: function() {
          const xAxis = d3.axisBottom(this.xScale).tickFormat(d => this.dateFormat(d));
          const yAxis = d3.axisLeft(this.yScale).ticks(5);
          d3.select(this.$refs.xAxis).call(xAxis);
          d3.select(this.$refs.yAxis).call(yAxis);
    },
    },
    mounted(){
        this.renderAxes()
    }
  };

  </script>

  <style lang="css">

    .tick text {
        font-size: 12px;
        text-transform: uppercase;
    }

    .x-axis path,
    .y-axis path {
        display: none;
    }

  </style>