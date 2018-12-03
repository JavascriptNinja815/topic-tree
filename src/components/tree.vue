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
      this.setData()
      this.drawElement()
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
      drawElement () {
        const g = d3.select(this.$el)
          .attr('transform', `translate(${ this.center.x }, ${ this.center.y })`)
          .attr('cursor', function () {
            return 'pointer'
          })
        this.drawPath(g)
        this.drawCircle(g)
        // this.drawText(g)
      },
      drawPath (ele) {
        const that = this
        ele.selectAll('line')
          .data(that.links, function (d) {
            return d
          })
          .enter().append('line')
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
        ele.selectAll('circle')
          .data(that.nodes, function (d) {
            return d
          })
          .enter().append('circle')
          .style('stroke', 'none')
          .style('fill', 'rgb(90, 228, 14)')
          
          .attr('r', function (d) {
            if (d.data.angle === null)
              return that.parentR
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
      }
    }
  }
</script>
<style>

</style>

