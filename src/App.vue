<template>
    <div id="app">
        <svg>
            <topic-tree :data="treeData"/>
        </svg>
    </div>
</template>

<script>
  import TopicTree from './components/tree'

  export default {
    name: 'app',
    components: { TopicTree },
    data: () => ({
      treeData: {
        name: 'Science',
        angle: null,
        isCollapsed: true,
        children: [
          {
            name: 'Physics',
            angle: 90,
            isCollapsed: true,
            hasChild: true,
            children: [{
              name: 'liner',
              angle: 20,
              isCollapsed: true,
              hasChild: true,
              children: [{
                name: 'motion',
                angle: 40,
                isCollapsed: true,
                hasChild: true,
                children: [{
                  name: 'color',
                  angle: 150,
                  isCollapsed: true,
                  hasChild: true
                }]
              }]
            }]
          },
          {
            name: 'Physics',
            angle: 180,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Math',
            angle: 270,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Biology',
            angle: 0,
            isCollapsed: true,
            hasChild: true
          }
        ]
      }
    }),
    mounted () {
      // this.getService()
    },
    methods: {
      // getService () {
      //   Service.getData().then(function (d) {
      //     this.treeData = d
      //
      //   })
      // }
    }
  }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
    }

    svg {
        width: 600px;
        height: 500px;
        background: grey;
    }
</style>
