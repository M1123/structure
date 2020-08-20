<template>
  <div class="hello" style="background-color: #ccc">
    <h3>Structure:</h3>
    <div v-if="flag">
      <div ref="div" id="div">
      </div>
    </div>
    <button @click="flag=!flag;">{{(flag)?'hide':'show'}}</button>
    <button @click="hi()">hi</button>
    <button @click="rec()">rec</button>
    <button @click="map(Data)">map</button>
  </div>
</template>

<script>
export default {
  props: ['Data','dictionary'],
  data() {
    return { 
      name: 'tree',
      flag: false,
    }
  },
  computed: {
  },
  mounted () {
  },
  methods: {
    hi () {
      console.log(this.Data);
      console.log(this.keys);

    },
    getkeys (obj) {
      let result = Object.keys(obj)
    },
    rec () {
      let queue = [];
      let values = [];
      queue.push(this.Data);
      while(queue.length > 0){
        let tempNode = queue.shift();
        values.push(tempNode.value);
        if (tempNode.left){
          queue.push(tempNode.left);
        }
        if (tempNode.right){
          queue.push(tempNode.right);
        }
      }
      console.log("rec -> values", values)
      return values;
    },
  },
}
</script>
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
