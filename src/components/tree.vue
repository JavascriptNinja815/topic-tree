<template>
    <g></g>
</template>

<script>
  import * as d3 from 'd3'

  export default {
    name: 'TopicTree',
    props: ['data','initPOS'],
    data: () => ({
      distance: 100,
      center: {
        x: 300,
        y: 250
      },
      childR: 20,
      parentR: 40,
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
      this.draw()
    },
    methods: {
      setData () {
        this.tree = d3.tree()
        this.root = d3.hierarchy(this.data)

        this.root.children.forEach(this.collapse)

        const treeData = this.tree(this.root)
        this.nodes = treeData.descendants()
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
      draw () {
        const that = this

        const g = d3.select(this.$el)
          .attr('transform', `translate(${ this.center.x }, ${ this.center.y })`)

        g.selectAll('circle')
          .data(that.nodes, function (d) {
            return d
          })
          .enter().append('circle')
          .style('stroke', 'green')
          .style('fill', 'blue')
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

