<template>
    <g></g>
</template>

<script>
  import * as d3 from 'd3'
  // import {TimelineMax} from 'gsap'

  export default {
    name: 'TopicTree',
    props: ['data'],
    data: () => ({
      distance: 200,
      duration: 750,
      fontSize: 20,
      childR: 18,
      parentR: 30,
      parentText: {
        x: 20,
        y: 15
      },
      childText: {
        x: 15,
        y: 10
      },
      center: {
        x: 500,
        y: 400
      },
      root: [],
      nodes: [],
      tree: null
    }),
    // watch: {
    //   data: {
    //     handler: function (newData) {
    //       console.log(newData)
    //     },
    //     deep: true
    //   }
    // },
    mounted () {
      //import data from topics.csv
      // d3.csv('topics.csv').then(function(data) {
      //   console.log(data)
      // })
      this.setData()
      this.calculateAngle()
      this.drawPanel()
    },
    methods: {
      setData () {
        this.tree = d3.tree()
        this.root = d3.hierarchy(this.data)
        this.root.children.forEach(this.collapse)

        const treeData = this.tree(this.root)
        this.nodes = treeData.descendants()        
        this.links = treeData.descendants().slice(1)
      },
      collapse (d) {
        if (d.children) {
          d._children = d.children
          d._children.forEach(this.collapse)
            d.children = null
        }
      },
      calculateAngle() {
        //calculate angle of each childrens
        this.nodes.forEach(function(d) {
          if (d.children && d.children.length) {
            let angle = 360 / d.children.length;
            d.children.forEach(function(ch, i) {
              ch.data.angle = 360 - angle * (i + 1);
            });
          }
        });
      },
      click (d) {
        if (d.children) {
          d._children = d.children
          d.children = null
        } else {
          d.children = d._children
          d._children = null;
        }
      },
      drawPanel () {
        const g = d3.select(this.$el)
          .attr('transform', `translate(${ this.center.x }, ${ this.center.y })`)
          .attr('cursor', 'pointer')
        this.drawPath(g)
        this.drawCircle(g)
        this.drawText(g)
      },
      drawPath (ele) {
        const that = this
        let duration = 0
        let delay = 0
        ele.selectAll('line')
          .data(that.links, function (d) {
            return d
          })
          .enter().append('line')
          .transition().duration(function () {
            duration = that.duration + delay + 300
            delay = delay + 250
            return duration
          })
          .attr('class', 'line')
          .attr('stroke', '#e0ddd5')
          .attr('stroke-width', '2px')
          .attr('x1', 0)
          .attr('y1', 0)
          .attr('x2', function (d) {
            const angle = (d.data.angle / 180) * Math.PI
            return that.distance * Math.sin(angle)
          })
          .attr('y2', function (d) {
            const angle = (d.data.angle / 180) * Math.PI
            return that.distance * Math.cos(angle)
          })
      },
      drawCircle (ele) {
        const that = this
        let duration = 0
        let delay = 0
        ele.selectAll('circle')
          .data(that.nodes, function (d) {
            return d
          })
          .enter().append('circle')
            .on('click', function (d){
              that.click(d)
            })
            .transition().duration(function () {
              duration = that.duration + delay
              delay = delay + 250
              return duration
            })
            .attr('class', 'circle')
            .attr('r', function (d) {
              if (d.data.angle === null) {
                return that.parentR
              }
              else {
                return that.childR
              }
            })
            .attr('cx', function (d) {
              if (d.data.angle === null) {
                return 0
              } else {
                const angle = (d.data.angle / 180) * Math.PI
                return that.distance * Math.sin(angle)
              }
            })
            .attr('cy', function (d) {
              if (d.data.angle === null) {
                return 0
              } else {
                const angle = (d.data.angle / 180) * Math.PI
                return that.distance * Math.cos(angle)
              }
            })
      },
      drawText (ele) {
        const that = this
        let duration = 0
        let delay = 0
        let textDistance = that.distance + 50
        ele.selectAll('text')
          .data(that.nodes, function (d) {
            return d
          })
          .enter().append('text')
            .attr('class', 'text')
            .html(function(d) {
                return d.data.name
              })
            .transition()
              .style('opacity', '1')
              .duration(function () {
                duration = that.duration + delay
                delay = delay + 250
                return duration
              })
              
              .attr('transform', function(d) {
                if (d.data.angle === null) {
                  return `translate(0, 0)`
                } else {
                  const angle = (d.data.angle / 180) * Math.PI
                  return `translate(${textDistance * Math.sin(angle)},${textDistance * Math.cos(angle)})`
                }
              })
      }
    }
  }
</script>
<style>
  .text div {
    width: 100%;
    height: 100%;
  }
  .text {
    fill: #434046;
    opacity: 0;
  }
  .circle {
    stroke: none;
    fill: #8ee000;
  }
</style>

