<template>
    <g></g>
</template>

<script>
  import * as d3 from 'd3'

  export default {
    name: 'TopicTree',
    props: ['data','initPOS'],
    data: () => ({
      distance: 200,
      duration: 750,
      parentText: {
        x: 20,
        y: 15
      },
      childText: {
        x: 15,
        y: 10
      },
      fontSize: 20,
      center: {
        x: 500,
        y: 400
      },
      childR: 18,
      parentR: 30,
      root: [],
      nodes: [],
      tree: null
    }),
    watch: {
      data: {
        handler: function (newData) {
          console.log(newData)
        },
        deep: true
      }
    },
    mounted () {
      //import data from topics.csv
      // d3.csv('topics.csv').then(function(data) {
      //   console.log(data)
      // })
      this.setData()
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
          if (d.height > 1) {
            d.children = null
          }
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
            duration = that.duration + delay + 250
            delay = delay + 250
            return duration
          })
          .attr('class', 'line')
          .attr('stroke', 'rgb(224, 221, 213)')
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
          .transition().duration(function () {
            duration = that.duration + delay
            delay = delay + 250
            return duration
          })
          .style('stroke', 'none')
          .style('fill', 'rgb(142, 224, 0)')
          
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
        ele.selectAll('.text')
          .data(that.nodes, function (d) {
            return d
          })
          .enter().append('g')
            .attr('transform', function(d) {
              if (d.data.angle === null) {
                return `translate(0, 0)`
              } else {
                const angle = (d.data.angle / 180) * Math.PI
                return `translate(${that.distance * Math.sin(angle)},${that.distance * Math.cos(angle)})`
              }
            })
            .attr('class', 'text')
            .append('foreignObject')
            .attr('width', function (d) {
              if (d.data.angel === null) {
                return that.parentText.x
              } else {
                return that.childText.x
              }
            })
            .attr('height', function (d) {
              if (d.data.angel === null) {
                return that.parentText.y
              } else {
                return that.childText.y
              }
            })
            .append('xhtml:div')
            .append('xhtml:p')
            .style('font-size', this.fontSize + 'px')
            .style('color', 'black')
            .html(function(d) {
              return d.data.name
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
</style>

