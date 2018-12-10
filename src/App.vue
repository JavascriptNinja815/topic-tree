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
        name: 'Parent',
        angle: null,
        isCollapsed: false,
        hasChild: true,
        children: [
          {
            name: 'child1',
            angle: null,
            isCollapsed: true,
            hasChild: true,
            children: [
              {
                name: 'Linear Motion ',
                angle: null,
                isCollapsed: true,
                hasChild: true,
                children: [
                  {
                    name: 'motion',
                    angle: null,
                    isCollapsed: true,
                    hasChild: true,
                    children: [
                        {
                        name: 'color1',
                        angle: null,
                        isCollapsed: true,
                        hasChild: false
                      }
                    ]
                  },
                  {
                    name: 'SUB-CATEGORY',
                    angle: null,
                    isCollapsed: true,
                    hasChild: true,
                    children: [
                      {
                        name: 'color',
                        angle: null,
                        isCollapsed: true,
                        hasChild: false
                      }
                    ]
                  }
                ]
              },
              {
                name: 'Linear Motion ',
                angle: null,
                isCollapsed: true,
                hasChild: true,
                children: [
                  {
                    name: 'motion',
                    angle: null,
                    isCollapsed: true,
                    hasChild: true,
                    children: [
                        {
                        name: 'color1',
                        angle: null,
                        isCollapsed: true,
                        hasChild: false
                      }
                    ]
                  },
                  {
                    name: 'SUB-CATEGORY',
                    angle: null,
                    isCollapsed: true,
                    hasChild: true,
                    children: [
                      {
                        name: 'color',
                        angle: null,
                        isCollapsed: true,
                        hasChild: false
                      }
                    ]
                  }
                ]
              }
            ]
          },
          {
            name: 'child2',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child3',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child4',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child5',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child6',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child7',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child8',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child9',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'child10',
            angle: null,
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
      width: 100%;
    }

    svg {
      width: 1000px;
      height: 800px;
      text-rendering: geometricPrecision;
      background: #f2f6f4;
  }
</style>
