<template>
  <svg class="sankey-chart"></svg>
</template>

<script>
import BaseChart from 'vue-d3-basechart'
import * as d3 from 'd3'
// import {drag} from 'd3-drag'
import {sankey} from 'd3-sankey'

export default BaseChart.extend({
  name: 'sankey-chart',
  props: ['width', 'height'],
  methods: {
    renderChart () {
      var units = 'Widgets'

      var margin = {top: 10, right: 10, bottom: 10, left: 10}
      var width = 700 - margin.left - margin.right
      var height = 300 - margin.top - margin.bottom

      var formatNumber = d3.format(',.0f')    // zero decimal places
      var format = (d) => formatNumber(d) + ' ' + units
      var color = d3.schemeCategory20

      // append the svg canvas to the page
      var svg = d3.select(this.$el)
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom)
        .append('g')
          .attr('transform',
                'translate(' + margin.left + ',' + margin.top + ')')

      // Set the sankey diagram properties
      var s = sankey()
          .nodeWidth(36)
          .nodePadding(40)
          .size([width, height])

      var path = s.link()

      // load the data
      d3.json('static/sankey-formatted.json', function (error, graph) {
        if (error !== null) {
          return error
        }

        s.nodes(graph.nodes)
            .links(graph.links)
            .layout(32)

        // add in the links
        var link = svg.append('g').selectAll('.link')
            .data(graph.links)
          .enter().append('path')
            .attr('class', 'link')
            .attr('d', path)
            .style('stroke-width', (d) => Math.max(1, d.dy))
            .sort((a, b) => b.dy - a.dy)

        // add the link titles
        link.append('title')
          .text((d) => (d.source.name + ' → ' +
                      d.target.name + '\n' + format(d.value)))

        // add in the nodes
        var node = svg.append('g').selectAll('.node')
          .data(graph.nodes)
          .enter().append('g')
            .attr('class', 'node')
            .attr('transform', (d) => 'translate(' + d.x + ',' + d.y + ')')
          // .call(drag()
          //   .origin((d) => d)
          //   .on('dragstart', function () {
          //     this.parentNode.appendChild(this)
          //   })
          //   .on('drag', dragmove))

        // add the rectangles for the nodes
        node.append('rect')
            .attr('height', (d) => d.dy)
            .attr('width', s.nodeWidth())
            .style('fill', function (d) {
              d.color = color(d.name.replace(/ .*/, ''))
            })
            .style('stroke', (d) => d3.rgb(d.color).darker(2))
          .append('title')
            .text((d) => d.name + '\n' + format(d.value))

        // add in the title for the nodes
        node.append('text')
            .attr('x', -6)
            .attr('y', (d) => d.dy / 2)
            .attr('dy', '.35em')
            .attr('text-anchor', 'end')
            .attr('transform', null)
            .text((d) => d.name)
          .filter((d) => (d.x < width / 2))
            .attr('x', 6 + s.nodeWidth())
            .attr('text-anchor', 'start')

        // // the function for moving the nodes
        // function dragmove (d) {
        //   d3.select(this).attr('transform',
        //       'translate(' + (
        //           d.x = Math.max(0, Math.min(width - d.dx, d3.event.x))
        //       ) + ',' + (
        //           d.y = Math.max(0, Math.min(height - d.dy, d3.event.y))
        //       ) + ')')
        //   s.relayout()
        //   link.attr('d', path)
        // }
      })
    }
  },
  watch: {
    width: 'renderChart',
    height: 'renderChart'
  }
})
</script>

<style lang="scss">
.sankey-chart {
  node rect {
    cursor: move;
    fill-opacity: .9;
    shape-rendering: crispEdges;
  }
  .node text {
    pointer-events: none;
    text-shadow: 0 1px 0 #fff;
  }
  .link {
    fill: none;
    stroke: #000;
    stroke-opacity: .2;
  }
  .link:hover {
    stroke-opacity: .5;
  }
}
</style>
