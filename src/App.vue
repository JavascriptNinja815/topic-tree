<template>
    <div id="app">
      <svg>
        <topic-tree :data="treeData"/>
      </svg>
      <svg>
        <old-tree :data="treeData"/>
      </svg>
    </div>
</template>

<script>
  import TopicTree from './components/tree'
  import OldTree from './components/tree_origin'

  export default {
    name: 'app',
    components: { TopicTree, OldTree },
    data: () => ({
      treeData: {
        name: 'Science',
        angle: null,
        cx: 0,
        cy: 0,
        isParent: true,
        hasChild: true,
        children: [
          {
            name: 'Physics',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true,
            children: [
              {
                name: 'Linear Motion 1',
                angle: null,
                cx: 0,
                cy: 0,
                isParent: true,
                hasChild: true,
                children: [
                  {
                    name: 'motion',
                    angle: null,
                    cx: 0,
                    cy: 0,
                    isParent: true,
                    hasChild: true,
                    children: [
                        {
                        name: 'color1',
                        angle: null,
                        cx: 0,
                        cy: 0,
                        isParent: true,
                        hasChild: false
                      }
                    ]
                  },
                  {
                    name: 'SUB-CATEGORY',
                    angle: null,
                    cx: 0,
                    cy: 0,
                    isParent: true,
                    hasChild: true,
                    children: [
                      {
                        name: 'color',
                        angle: null,
                        cx: 0,
                        cy: 0,
                        isParent: true,
                        hasChild: false
                      }
                    ]
                  }]
              },
              {
                name: 'Linear motion 2',
                angle: null,
                cx: 0,
                cy: 0,
                isParent: true,
                hasChild: true,
                children: null
              },
              {
                name: 'Linear motion 3',
                angle: null,
                cx: 0,
                cy: 0,
                isParent: true,
                hasChild: true,
                children: null
              }
            ]
          },
          {
            name: 'Chemistry',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Physical Science',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Biology',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Anatomy',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Physiology',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Zoology',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Science',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Physics - Intermediate',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
            hasChild: true
          },
          {
            name: 'Biology',
            angle: null,
            cx: 0,
            cy: 0,
            isParent: true,
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
  // export default {

  // }
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
