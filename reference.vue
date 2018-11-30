<template>
  <div id='body'></div>
</template>

<script>
  import * as d3 from 'd3'
  import numeral from 'numeral'

  export default {
    name: 'OrgChart',
    props: ['data', 'search', 'selected'],
    data: () => ({
      opt: {
        duration: 750,
        rectW: 190, /* Width of the rectangle */
        rectH: 50, /* Height of the rectangle */
        rectSpacing: 20, /* Spacing between the rectangles */
        fixedDepth: 90 /* Height of the line for child nodes */
      },
      values: null,
      counter: 0,
      root: null,
      nodes: null,
      tree: null,
      diagram: null,
      colorPatten: d3.scaleLinear().domain([0, 100]).range(['#464E4D', '#B49F63']).interpolate(d3.interpolateRgb),
      timer: null,
      selectedNode: null
    }),
    watch: {
      search (keyword) {
        const that = this
        clearTimeout(this.timer)
        this.timer = setTimeout(function () {
          keyword.length > 0 ? that.getChildrenMatchingSearch(that.root, keyword) : that.resetData(that.root)
          that.update(that.root)
        }, 250)
      },
      selected (val) {
        if (val === null) {
          this.selectedNode = null
        }
        this.update(this.root)
      }
    },
    mounted () {
      this.draw()
    },
    methods: {
      draw () {
        const that = this
        d3.select(this.$el).selectAll('*').remove()
        const width = this.$el.offsetWidth
        const height = this.$el.offsetHeight
        const childWidth = parseInt((this.data.children.length * this.opt.rectW) / 2)

        const svg = d3.select(this.$el).append('svg').attr('class', 'enterpriseBody').attr('width', '100%').attr('height', '100%')
        this.diagram = svg.append('g')
          .attr('transform', 'translate(' + parseInt(childWidth + ((width - childWidth * 2) / 2)) + ', 20)')
        const legend = svg.append('g').attr('class', 'legend')
        const zoom = d3.zoom().scaleExtent([0.15, 3]).on('zoom', function () {
          that.diagram.attr('transform', d3.event.transform)
        })
        const transform = d3.zoomIdentity.translate(parseInt(childWidth + ((width - childWidth * 2) / 2)), 20).scale(1)

        this.tree = d3.tree().nodeSize([this.opt.rectW + this.opt.rectSpacing, this.opt.rectH + this.opt.rectSpacing])
        this.root = d3.hierarchy(this.data, function (d) {
          return d.children
        })
        svg.call(zoom.transform, transform)

        this.root.x0 = 0
        this.root.y0 = height / 2

        // Collapse after the second level
        this.root._backupChildren = this.root.children
        this.root.children.forEach(this.collapse)
        this.update(this.root)
        this.drawLegend(legend)
        svg.call(zoom)
      },
      drawLegend (ele) {
        const elem = ele.append('foreignObject')
          .attr('x', 0)
          .attr('y', 10)
          .attr('width', 200)
          .attr('height', 20)
          .append('xhtml:div')
          .attr('class', 'legend')
        elem.append('xhtml:span')
          .attr('class', 'legend-start')
          .text('0%')
        elem.append('xhtml:span')
          .attr('class', 'legend-center')
          .text('50%')
        elem.append('xhtml:span')
          .attr('class', 'legend-end')
          .text('100%')
      },
      getCorrectWidth (widths) {
        const nodeWidth = parseInt((this.data.children.length * this.opt.rectW) / 2) + 40
        let count = 0
        const minWidth = nodeWidth * widths.length
        let maxWidth = widths.reduce((cur, pre) => {
          if (pre > nodeWidth) {
            cur += pre
            count++
          }
          return cur
        }, 0)
        if (count <= 1 || minWidth >= maxWidth) {
          return minWidth
        }
        if (widths[0] > nodeWidth) {
          maxWidth -= widths[0] / 2 - nodeWidth / 2
        }
        if (widths[widths.length - 1] > nodeWidth) {
          maxWidth -= widths[widths.length - 1] / 2 - nodeWidth / 2
        }
        return maxWidth
      },
      getNodeWidth (node) {
        const nodeWidth = parseInt((this.data.children.length * this.opt.rectW) / 2) + 40
        if (!node.children || node.data.childLayout === 'vertical') return nodeWidth
        const widths = node.children.map((d, i) => {
          return this.getNodeWidth(d)
        })
        return this.getCorrectWidth(widths)
      },
      adjust (nodes) {
        const nodeWidth = parseInt((this.data.children.length * this.opt.rectW) / 2) + 40
        return nodes.map(d => {
          let width = this.getNodeWidth(d)
          if (d.children) {
            let sPos = d.x + nodeWidth / 2 - width / 2
            const widths = d.children.map((c, i) => {
              let width = this.getNodeWidth(c)
              return c.children && c.children.length > 1 ? width / 2 + nodeWidth / 2 : width
            })
            let count = widths.reduce((s, c) => c > nodeWidth ? s + 1 : s, 0)
            d.children = d.children.map((c, i) => {
              c.x = sPos
              c.x0 = sPos
              const childWidth = widths[i] + nodeWidth
              sPos += count > 1 ? childWidth : nodeWidth
              return c
            })
          }
          return d
        })
      },
      update (source) {
        const that = this
        let treeData = this.tree(this.root)
        this.nodes = treeData.descendants()
        const links = treeData.links(this.nodes)

        let depth = 0
        let counter = 0
        let posX = 0
        this.nodes.forEach(function (d) {
          if (!that.selectedNode) (d.data.isSelected = false)
          if (d.parent && d.parent.data && d.parent.data.childLayout && d.parent.data.childLayout === 'vertical') {
            if (depth === d.depth && posX === d.parent.x) {
              counter++
            } else {
              depth = d.depth
              counter = 0
            }
            d.x = d.parent.x
            posX = d.parent.x
          }

          d.y = d.depth * that.opt.fixedDepth + counter * that.opt.fixedDepth
          d.x0 = d.x
          d.y0 = d.y
        })

        // adjust x position
        this.nodes = this.adjust(this.nodes)

        const node = this.diagram.selectAll('g.node')
          .data(that.nodes, function (d) {
            return d.id || (d.id = ++that.counter)
          })

        // Enter any new modes at the parent's previous position.
        const nodeEnter = node.enter().append('g')
          .attr('class', 'node')
          .attr('transform', function () {
            return 'translate(' + source.x0 + ',' + source.y0 + ')'
          })

        // Add Circle for the nodes
        const foreignObject = nodeEnter
          .append('foreignObject')
          .attr('width', this.opt.rectW)
          .attr('height', this.opt.rectH)
        foreignObject.append('xhtml:div')
          .attr('class', function (d) {
            if (d.isSearched) {
              return 'node searched'
            } else {
              return 'node'
            }
          })
          .html(function (d) {
            return that.makeNode(d)
          })
          .on('click', function (d) {
            that.selectNode(d, this)
          })
        foreignObject.append('xhtml:div')
          .attr('class', 'expand_collapse_icon')
          .style('display', function (d) {
            return (d.children || d._children) ? 'block' : 'none'
          })
          .html(function (d) {
            let icon = 'fal fa-minus'
            if (d.children === null) {
              icon = 'fal fa-plus'
            }
            if (d.children || d._children) {
              return '<i class="' + icon + '"></i>'
            }
          })
          .on('click', function (d) {
            that.nodeClick(d, this)
          })

        // Transition to the proper position for the node
        const nodeUpdate = nodeEnter.merge(node)
        nodeUpdate.select('div')
          .attr('class', function (d) {
            if (d.data.isSelected) {
              return 'node selected'
            } else if (d.isSearched) {
              return 'node searched'
            } else {
              return 'node'
            }
          })
          .select('.expand_collapse_icon')
          .style('display', function (d) {
            return (d.children || d._children) ? 'block' : 'none'
          })
          .html(function (d) {
            let icon = 'fal fa-minus'
            if (d.children === null) {
              icon = 'fal fa-plus'
            }
            if (d.children || d._children) {
              return '<i class="' + icon + '"></i>'
            }
          })

        nodeUpdate.transition()
          .duration(this.opt.duration)
          .attr('transform', function (d) {
            if (d.parent && d.parent.data && d.parent.data.childLayout && d.parent.data.childLayout === 'vertical') {
              return 'translate(' + d.parent.x + ',' + d.y + ')'
            } else {
              return 'translate(' + d.x + ',' + d.y + ')'
            }
          })

        // Remove any exiting nodes
        node.exit().transition()
          .duration(this.opt.duration)
          .attr('transform', function (d) {
            return 'translate(' + source.x + ',' + source.y + ')'
          })
          .remove()

        this.drawLink(links, source)
      },
      makeNode (info) {
        let icon = 'fab fa-amazon'
        if (info.data.icon === 'cart') {
          icon = 'fal fa-shopping-cart'
        } else if (info.data.icon === 'web') {
          icon = 'fal fa-browser'
        } else if (info.data.icon === 'mobile') {
          icon = 'fal fa-mobile-alt'
        } else {
          icon = 'fab fa-amazon'
        }

        const weight = parseInt(info.data['top-value'].replace('%', ''))

        return '<div class="node-left">' +
          '<div class="type_icon">' +
          '<i class="' + icon + '"></i>' +
          '</div>' +
          '<p class="node_desc">' + info.data.desc + '</p>' +
          '</div>' +
          '<div class="node-right" style="background-color: ' + this.colorPatten(weight) + '">' +
          '<div class="val">' + this.formatPercentage(info.data['top-value']) + '</div>' +
          '<div class="val sub-value">' + this.formatNumberSign(info.data['bottom-value']) + '</div>' +
          '</div>'
      },
      drawLink (links, source) {
        const that = this
        const link = this.diagram.selectAll('path.link')
          .data(links, function (d) {
            return d.target.id
          })
        const lineFunction = d3.line()
          .x(function (d) {
            return d.x
          })
          .y(function (d) {
            return d.y
          })
          .curve(d3.curveLinear)
        const linkEnter = link.enter().append('path', 'g')
          .attr('class', 'link')
          .attr('d', function (d) {
            const line = (function (d) {
              let lineData = null
              if (d.source.data.childLayout && d.source.data.childLayout === 'vertical') {
                lineData = [{
                  'x': source.x0 + parseInt(that.opt.rectW / 2),
                  'y': source.y0 + that.opt.rectH
                },
                  { 'x': source.x0 + parseInt(that.opt.rectW / 2), 'y': source.y0 + that.opt.rectH },
                  { 'x': source.x0 + parseInt(that.opt.rectW / 2), 'y': source.y0 + that.opt.rectH },
                  { 'x': source.x0 + parseInt(that.opt.rectW / 2), 'y': source.y0 + that.opt.rectH },
                  { 'x': source.x0 + parseInt(that.opt.rectW / 2), 'y': source.y0 + that.opt.rectH }]
              } else {
                lineData = [{
                  'x': source.x0 + parseInt(that.opt.rectW / 2),
                  'y': source.y0 + that.opt.rectH
                },
                  { 'x': source.x0 + parseInt(that.opt.rectW / 2), 'y': source.y0 + that.opt.rectH },
                  { 'x': source.x0 + parseInt(that.opt.rectW / 2), 'y': source.y0 + that.opt.rectH },
                  { 'x': source.x0 + parseInt(that.opt.rectW / 2), 'y': source.y0 + that.opt.rectH }]
              }
              return lineData
            })(d)
            return lineFunction(line)
          })
        // Transition links to their new position.
        const linkUpdate = linkEnter.merge(link)
        linkUpdate.transition()
          .duration(this.opt.duration)
          .attr('d', function (d) {
            const line = (function (d) {
              let lineData = null
              if (d.source.data.childLayout && d.source.data.childLayout === 'vertical') {
                lineData = [
                  { 'x': d.source.x + parseInt(that.opt.rectW / 2), 'y': d.source.y + that.opt.rectH },
                  { 'x': d.source.x + parseInt(that.opt.rectW / 2), 'y': d.source.y + that.opt.rectH + 20 },
                  { 'x': d.source.x - 20, 'y': d.source.y + that.opt.rectH + 20 },
                  { 'x': d.source.x - 20, 'y': d.target.y + parseInt(that.opt.rectH / 2) },
                  { 'x': d.source.x, 'y': d.target.y + parseInt(that.opt.rectH / 2) }
                ]
              } else {
                lineData = [
                  { 'x': d.source.x + parseInt(that.opt.rectW / 2), 'y': d.source.y + that.opt.rectH },
                  { 'x': d.source.x + parseInt(that.opt.rectW / 2), 'y': d.target.y - 20 },
                  { 'x': d.target.x + parseInt(that.opt.rectW / 2), 'y': d.target.y - 20 },
                  { 'x': d.target.x + parseInt(that.opt.rectW / 2), 'y': d.target.y }
                ]
              }
              return lineData
            })(d)
            return lineFunction(line)
          })
        // Transition exiting nodes to the parent's new position.
        link.exit().transition()
          .duration(this.opt.duration)
          .attr('d', function (d) {
            /* This is needed to draw the lines right back to the caller */
            const line = (function (d) {
              let lineData = null
              if (d.source.data.childLayout && d.source.data.childLayout === 'vertical') {
                lineData = [
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH },
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH },
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH },
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH },
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH }
                ]
              } else {
                lineData = [
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH },
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH },
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH },
                  { 'x': source.x + parseInt(that.opt.rectW / 2), 'y': source.y + that.opt.rectH }
                ]
              }
              return lineData
            })(d)
            return lineFunction(line)
          })
      },
      selectNode (d) {
        if (this.selectedNode) {
          this.selectedNode.data.isSelected = false
        }
        d.data.isSelected = true
        this.selectedNode = d
        this.$emit('select', this.selectedNode.data)
        this.update(d)
      },
      nodeClick (d, ele) {
        if (d.children) {
          d._children = d.children
          d.children = null
          d3.select(ele).html('<i class="fal fa-plus"></i>')
        } else {
          d.children = d._children
          d._children = null
          d3.select(ele).html('<i class="fal fa-minus"></i>')
        }
        this.update(d)
      },
      collapse (d) {
        if (d.children) {
          d._children = d.children
          d._backupChildren = d.children
          d._backupChildren.forEach(this.collapse)
          if (d.depth > 1) {
            d.children = null
          }
        }
      },
      resetData (d) {
        d.isSearched = false
        if (d._backupChildren) {
          d.children = d._backupChildren
          d._children = null
          d._backupChildren.forEach(this.resetData)
          if (d.depth > 1) {
            d._children = d._backupChildren
            d.children = null
          }
        }
      },
      getChildrenMatchingSearch (node, search) {
        const that = this
        node.isSearched = this.isMatchingNode(node, search)

        if (node._backupChildren) {
          let children = []
          node.children = null
          node._children = null
          node._backupChildren.forEach(function (d) {
            let found = that.getChildrenMatchingSearch(d, search)
            if (found) {
              children.push(found)
            }
          })

          if (children.length > 0) {
            node.children = children
            return node
          }
        }

        return (node.children || this.isMatchingNode(node, search)) ? node : null
      },
      isMatchingNode (node, search) {
        if ((node.data.desc).toLowerCase().indexOf(search) > -1) {
          return true
        } else {
          return false
        }
      },
      formatPercentage: function (value) {
        return numeral(parseInt(value.replace('%', ''))).format('0,0') + '%'
      },
      formatNumberSign: function (value) {
        return '$ ' + numeral(value).format('0.0 a')
      }
    }
  }
</script>

<style>
  .node {
    cursor: default;
    border: 1px solid white;
    border-radius: 4px;
    background: #1c252c;
    width: 190px;
    height: 50px;
    display: flex;
  }

  .node:hover {
    border: 1px solid #b49f63;
  }

  .node.searched {
    border: 2px solid yellow;
  }

  .node.selected, .node.selected .type_icon, .node.selected:hover .type_icon {
    border: 1px solid red;
  }

  .node-left {
    width: 65%;
    position: relative;
    color: white;
    border-radius: 4px 0 0 4px;
  }

  .type_icon {
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    top: -16px;
    left: 15px;
    border-radius: 50%;
    border: 1px solid white;
    background: #1c252c;
    width: 36px;
    height: 36px;
    font-size: 18px;
  }

  .node:hover .type_icon {
    border: 1px solid #b49f63;
    color: #b49f63;
  }

  .node_desc {
    position: relative;
    bottom: 13px;
    margin: 0;
    text-align: left;
    padding: 0 5px;
  }

  .node-right {
    flex: 1 1 0;
    border-radius: 0 3px 3px 0;
  }

  .val {
    margin: 3px;
  }

  .sub-value {
    font-size: 12px;
    text-transform: uppercase;
  }

  .link {
    fill: none;
    stroke: #eeeeee;
  }

  #body {
    cursor: move;
    height: calc(100% - 32px);
  }

  .legend {
    width: 100%;
    height: 100%;
    display: flex;
    position: relative;
    background: #464E4D; /* For browsers that do not support gradients */
    background: linear-gradient(to right, #464E4D, #B49F63); /* Standard syntax (must be last) */
    border: 1px solid #eeeeee;
  }

  .expand_collapse_icon {
    width: 20px;
    height: 20px;
    border: 1px solid #eeeeee;
    border-radius: 50%;
    background: #1c252c;
    color: white;
    position: absolute;
    left: calc(50% - 10px);
    bottom: -10px;
    cursor: pointer;
  }

  .legend-start, .legend-center, .legend-end {
    position: absolute;
  }

  .legend-start {
    left: 0;
  }

  .legend-center {
    left: 50%;
    transform: translate(-50%, 0);
  }

  .legend-end {
    right: 0;
  }
</style>
