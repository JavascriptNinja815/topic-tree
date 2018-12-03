<template>
  <div id="topic-tree">
  </div>
</template>

<script>
import * as d3 from 'd3'

// const props = {
//   data: Object,
//   duration: {
//     type: Number,
//     default: 750
//   },
//   marginX: {
//     type: Number,
//     default: 20
//   },
//   marginY: {
//     type: Number,
//     default: 20
//   },
//   nodeText: {
//     type: String,
//     required: true
//   },
//   identifier: {
//     type: Function,
//     default: () => i++
//   },
//   zoomable: {
//     type: Boolean,
//     default: false
//   },
//   radius: {
//     type: Number,
//     default: 3
//   }
// }
export default {
  name: 'TopicTreeBackup',
  props: ['data'],
  data: () => ({
    treeData: {
      "name": "Science",
      "children": [
        {
          "name": "Physics"
        },{
          "name": "Biology"
        },{
          "name": "Chemistry"
        },{
          "name": "Mathematics"
        }
      ]
    },
    root: null
  }),

  mounted () {
    this.draw()
  },
  methods: {
    draw () {
      //initialize
      // d3.select("#topic-tree").selectAll('*').remove()

      const margin = {top: 20, right: 90, bottom: 30, left: 90}
      const width = 960 - margin.left - margin.right
      const height = 500 - margin.top - margin.bottom

      const svg = d3.select("#topic-tree").append("svg")
          .attr("class", "svg-container").attr("width", '100%').attr("height", '100%')
      this.diagram = svg.append("g")
          .attr("transform", "translate(" + width / 2 + "," + height / 2 + ")")
          .attr("background-color", 'lightblue')

      this.i = 0
      this.duration = 750

      // declares a tree layout and assigns the size
      this.treemap = d3.tree().size([height, width]);

      // Assigns parent, children, height, depth
      this.root = d3.hierarchy(this.treeData, function(d) {
        return d.children
      });
      this.root.x0 = width / 2;
      this.root.y0 = height / 2;

      this.update(this.root);

    },
    update (source) {
      const that = this
      // Assigns the x and y position for the nodes
      let treeData = this.treemap(this.root);
      console.log(treeData)

      // Compute the new tree layout.
      this.nodes = treeData.descendants()
      const links = treeData.descendants().slice(1);

      // Normalize for fixed-depth.
      this.nodes.forEach(function(d){
        d.y = d.depth * (100 + depth)
      });

      // ****************** Nodes section ***************************

      // Update the nodes...
      const node = this.diagram.selectAll('g.node')
          .data(that.nodes, function(d) {return d.id || (d.id = ++this.i); });

      // Enter any new modes at the parent's previous position.
      const nodeEnter = node.enter().append('g')
          .attr('class', 'node')
          .attr("transform", function() {
            return "translate(" + source.y0 + "," + source.x0 + ")";
        })
        .on('click', function (d) {
          that.selectNode(d)
        })
      // Add Circle for the nodes
      nodeEnter.append('circle')
          .attr('class', 'node')
          .attr('r', 1e-6)
          .style("fill", "rgb(90, 228, 14)");

      // Add labels for the nodes
      nodeEnter.append('text')
          .attr("dy", ".35em")
          .attr("x", function(d) {
              return d.children || d._children ? -13 : 13;
          })
          .attr("text-anchor", function(d) {
              return d.children || d._children ? "end" : "start";
          })
          .text(function(d) { return d.data.name; });

      // UPDATE
      const nodeUpdate = nodeEnter.merge(node);

      // Transition to the proper position for the node
      nodeUpdate.transition()
        .duration(this.duration)
        .attr("transform", function(d) {
            return "translate(" + d.y + "," + d.x + ")";
        });

      // Update the node attributes and style
      nodeUpdate.select('circle.node')
        .attr('r', function(d) {
          return d._children ? 10 : 20;
        })
        .style("fill", "rgb(90, 228, 14)")
        .attr('cursor', 'pointer');

      // Remove any exiting nodes
      let nodeExit = node.exit().transition()
          .duration(this.duration)
          .attr("transform", function() {
              return "translate(" + source.y + "," + source.x + ")";
          })
          .remove();

      // On exit reduce the node circles size to 0
      nodeExit.select('circle')
        .attr('r', 1e-6);

      // On exit reduce the opacity of text labels
      nodeExit.select('text')
        .style('fill-opacity', 1e-6);

      // ****************** links section ***************************

      // Update the links...
      let link = this.diagram.selectAll('path.link')
          .data(links, function(d) { return d.id; });

      // Enter any new links at the parent's previous position.
      let linkEnter = link.enter().insert('path', "g")
          .attr("class", "link")
          .attr('d', function(){
            let o = {x: source.x0, y: source.y0}
            return diagonal(o, o)
          });

      // UPDATE
      let linkUpdate = linkEnter.merge(link);

      // Transition back to the parent element position
      linkUpdate.transition()
          .duration(this.duration)
          .attr('d', function(d){ return diagonal(d, d.parent) });

      // Remove any exiting links
      link.exit().transition()
          .duration(this.duration)
          .attr('d', function() {
            let o = {x: source.x, y: source.y}
            return diagonal(o, o)
          })
          .remove();

      // Store the old positions for transition.
      this.nodes.forEach(function(d){
        d.x0 = d.x;
        d.y0 = d.y;
      });

      // Creates a line (diagonal) path from parent to the child nodes
      function diagonal(s, d) {
        let path = `M ${s.y} ${s.x}
                    L ${d.y} ${d.x}`
        return path
      }
    },

    // Toggle children on click.
    selectNode(d) {
      if (d.children) {
        d._children = d.children;
        d.children = null;
      } else {
        d.children = d._children;
        d._children = null;
      }
      this.update(d);
    }
  }
}
</script>
<style>
  .node {
    cursor: pointer;
  }
  .node circle {
    fill: rgb(90, 228, 14);
    fill-opacity: 0.7;
  }
  .node text {
    font: 12px sans-serif;
  }
  .link {
    fill: lightgrey;
    /* fill: none; */
  }
  .svg-container {
    background: #2E3C48;
    min-width: 988px;
    min-height: 726px;
    height: 100%;
  }
</style>

