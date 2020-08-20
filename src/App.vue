<template>
  <div id="app">
    <div class="task">
      <h1>Тестовое задание</h1>
      <p>
        Из полей "value" приложенной структуры создать объект, пригодный на Ваш взгляд, 
        для хранения иерархических данных и отрисовать их, как вложенный список.
      </p>
    </div>
    <div class="container">
      <template v-if="fileData!=null">
        <div class="col">
          <p>Записать объект для хранения (Объект ID)</p>
          <button @click="writeRes(result, dictionary, 'файл для хранения')">Скачать</button>
        </div>
        <div class="col">
          <p>Записать объект для хранения с возможностью восстановления массива исходных значений (массивы ID)</p>
          <button @click="writeRes(ids, dictionary, 'файл для хранения ')">Скачать</button>
        </div>
      </template>
      <template v-else>
        <div class="col">
          <p>Приложите файл json (массив объектов с ключом "data") для формирования объекта для хранения</p>
          <form>
            <input type="file" name="fileinput" accept=".json" id="fileinput" @change="importFile">
          </form>
        </div>
        <div class="col">
          <p>Приложите файл json (массив id с ключом "structure" и массивом значений с ключом "dictionary")
            для восстановления массива исходных значений
          </p>
          <form>
            <input type="file" name="fileinput2" accept=".json" id="fileinput2" @change="importFile2">
          </form>
        </div>
      </template>
    </div>
    <div id="structureDiv"></div>
  </div>
</template>

<script>
import tree from './components/tree.vue'
// import fileData from "./map.json";

export default {
  data() {
    return {
      fileData: null, //данные из файла
      limit: undefined,
      name: 'App',
      result: {},
      dict: {},
      maxlen:0,
      dictionary: [],
      file: null,
    }
  },
  computed: {
    ids () {
    return this.idStructure(this.fileData.data,this.limit);
    },
  },
  mounted () {},
  methods: {
    importFile (e) {
      let file = e.target.files[0] || e.dataTransfer.files[0];
      let input = document.getElementById('fileinput');
      let reader = new FileReader();
      reader.readAsText(file);
      reader.onload = ()=> {
        this.fileData = JSON.parse(reader.result);
        this.hi(this.ids,this.dictionary);
      };
    },
    importFile2 (e) {
      let file = e.target.files[0] || e.dataTransfer.files[0];
      let input = document.getElementById('fileinput');
      let reader = new FileReader();
      reader.readAsText(file);
      reader.onload = ()=> {
        let fileData = JSON.parse(reader.result);
        this.restore(fileData);
      };
    },
    writeRes(structure, dictionary, name) { //запись в файл
      let obj = (dictionary == null) ? {
        structure,
        dictionary
      } : structure
      let data = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(obj));
      let link = document.createElement('a');
      link.setAttribute('href',data);
      link.setAttribute('download',name||'file'+'.json');
      link.click();
    },
    hi (ids, dictionary) { //формирование и отрисовка структуры в DOM
        for (let i = 0; i < ids.length; i++) {
          const element = ids[i];
          let parent = this.result;
          let level = 0;
          writeChildren(element,level,parent)
        }
        function writeChildren(arr, level, parent) {
          let prop = arr[level-1]
          let div = document.getElementById('structureDiv')
          let id = arr.slice(0,level);
          if (!parent.hasOwnProperty(prop)&& id!=undefined){
            parent[prop]={};
            let parentDiv = document.getElementById(arr.slice(0,level-1))||div;
            if (level === 0) parentDiv = div;
            let ul = document.createElement('ul');
            if (level>1) ul.className='hidden'
            ul.id = id;
            let li = document.createElement('li');
            li.textContent = dictionary[prop]||prop;
            li.className='structure'
            li.onclick = (li) => {
              for (let node of li.target.parentElement.children) {
                if (node.localName == 'ul') node?.classList?.toggle("hidden");
                if (node.localName == 'li') node?.classList?.toggle("active");
              }
            },
            ul.append(li);
            parentDiv.append(ul);
            parentDiv = document.getElementById(arr.slice(0,level-1))||div;
          }
          level++;
          if (level<=arr.length) writeChildren(arr, level, parent[prop])
        }
    },

    restore (fileData) { //восстановление массива исходных значений
      if (fileData==null || fileData.structure == null || fileData.dictionary == null) {
        alert('Неверный формат файла');
        return;
      }
      let structure = fileData.structure;
      let dictionary = fileData.dictionary;
      this.hi(structure, dictionary)
      let result = {data:[]};
      structure.forEach(el => {
        let str = '';
        for (let i = 0; i < el.length; i++) {
          const element = dictionary[el[i]];
          if (i < el.length) element += `\\`
          str += element
        }
        result.data.push(str)
      });
      this.writeRes(result, null, 'Массив значений.json')

    },

    idStructure (stucture, limit) { //формирование массивов идентификаторов
      let List = (limit!=null) ? stucture.slice(0,limit) : stucture;
      this.maxlen = 0;
      let resArr = [];
      let valueId = 0;
      console.table("idStructure -> List", List)
      for (let k = 0; k < List.length; k++) {
        let el = List[k];
        el.splitedKey = el.key?.split(`\\`);
        el.splitedVal = el.value?.split(`\\`);
        el.value = [];
        if (el.splitedVal.length > this.maxlen) this.maxlen = el.splitedVal?.length||0;
        for (let i = 0; i < el.splitedVal?.length||0; i++) {
          if (!this.dict.hasOwnProperty(el.splitedKey[i])){
            this.dict[el.splitedKey[i]] = {
              name:el.splitedVal[i],
              id: valueId,
            };
            valueId++;
          }
          el.value[i] = this.dict[el.splitedKey[i]].id;
        }
        List[k] = (el.value);
      }
      this.getDictionary();
      console.table('Массивы id: ', List);
      console.log("глубина вложенности - ", this.maxlen)
      return List;
    },

    getDictionary () { // формирование справочника с уникальными значениями
      let obj = this.dict;
      let result = [];
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          const id = obj[key].id;
          const name = obj[key].name;
          result[id] = name;
        }
      }
      this.dictionary = result;
      return result;
    },

  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.structure {
  text-align: start;
  cursor: pointer;
  list-style-type: none;
  background-color: #b1b1b1;
  border-radius: 3px;
  padding: 15px 10px;
}
.structure:before {
  content:url('./images/plus-circle-outline.png');
  display: inline-block;
  height: 100%;
  vertical-align: middle;
}
.structure:hover {
  color: red;
}
.hidden {display: none;}
.active {
  background-color: #b4dba4;
}
.structure.active:before {
  content:url('./images/minus-circle-outline.png');
}
.container {
  display: flex;
  justify-content: space-around;
  height: 80%;
  margin: auto;
}
.col {
  min-width: 200px;
  min-height: 50px;
  background-color: #b1b1b1;
  border: black 1px solid;
}
</style>
