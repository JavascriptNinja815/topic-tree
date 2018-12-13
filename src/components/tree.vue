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
      duration: 1000,
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
      tree: null,
      panel: null,
      selectNode: null
    }),
    mounted () {
      this.draw()
    },
    methods: {
      draw () {
        this.tree = d3.tree()
        this.root = d3.hierarchy(this.data, function(d){
          return d.children
        })
        this.root.children.forEach(this.collapse)
        this.panel = d3.select(this.$el)
          .attr('transform', `translate(${ this.center.x }, ${ this.center.y })`)
          .attr('cursor', 'pointer')
        this.update(this.root)
      },
      collapse (d) {
        if (d.children) {
          d._children = d.children
          d._children.forEach(this.collapse)
            d.children = null
        }
      },
      update (source) {
        const that = this
        let duration = 0
        let delay = 0
        const treeData = this.tree(source)
        this.nodes = treeData.descendants()        
        const links = treeData.descendants().slice(1)
        //calculate (x, y position using angle)
        this.nodes.forEach(function(d) {
          if (d.children && d.children.length) {
            let angle = 360 / d.children.length;
            d.children.forEach(function(ch, i) {
              ch.data.angle = 360 - angle * (i + 1);
              // ch.data.cx = that.distance * Math.sin(d.data.angle * Math.PI / 180)
              // ch.data.cy = that.distance * Math.cos(d.data.angle * Math.PI / 180)
            });
            
          }
        });
        console.log(links)
        this.drawLink(links)
        const node = this.panel.selectAll('g.node')
          .data(that.nodes, function (d) {
            return d.id || (d.id = ++that.counter)
          })
        //draw circle
        node.enter().append('g')
          .attr('class', 'node')
          .append('circle')
            .on('click', function (d) {
                return that.nodeClick(d, this)
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
          
          // .on('click', function (d) {
          //   that.selectNode(d, this)
          // })

        this.drawText()
      },
      nodeClick (d, ele) {
        let that = this
        let duration = 0
        let delay = 0
        //move clicked node to other position based on it's angle
        d3.select(ele).transition().duration(function () {
          duration = that.duration + delay
          delay = delay + 250
          return duration
        })
        .attr('transform', function(d) {
          if (d.data.angle === null) {
            return `translate(0, 0)`
          } else {
            const angle = (d.data.angle / 180) * Math.PI
            return `translate(${that.distance * 1.5 * Math.sin(angle)},${that.distance * 1.5 * Math.cos(angle)})`
          }
        })
        if (d.children) {
          d._children = d.children
          d.children = null
        } else {
          d.children = d._children
          d._children = null;
        }
        this.drawLink(d)
        this.update(d)
      },
      drawLink (links) {
        const that = this
        let duration = 0
        let delay = 0
        this.panel.selectAll('line')
          .data(links, function (d) {
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
      drawText () {
        const that = this
        let duration = 0
        let delay = 0
        let textDistance = that.distance + 50
        this.panel.selectAll('text')
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

