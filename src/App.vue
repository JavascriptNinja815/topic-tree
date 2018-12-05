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
        isCollapsed: false,
        children: [
          {
            name: 'Physics',
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
                    children: [{
                      name: 'color',
                      angle: null,
                      isCollapsed: true,
                      hasChild: false
                    }]
                  }]
              }
            ]
          },
          {
            name: 'Chemistry',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Physical Science',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Biology',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Anatomy',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Physiology',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Zoology',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Science',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Physics - Intermediate',
            angle: null,
            isCollapsed: true,
            hasChild: true
          },
          {
            name: 'Biology',
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
