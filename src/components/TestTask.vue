<template>
  <div class="container">
    {{ loadMsg }}
    <div class="flexbox-wrapper">
    <template v-for="(array, index) in typedArray">
      <div class="input-wrapper multiple__selecting" :key='index'>
        <label class="typo__label">{{ "Тип " + arrayOfTypes[index] }}</label>
        <Multiselect v-if="arrayOfTypes[index] !== 'object'"
                    v-model="value[arrayOfTypes[index]]" :options="array" 
                    :multiple="true" 
                    :close-on-select="true" 
                    :clear-on-select="false" 
                    :preserve-search="true" 
                    placeholder="Выберите значение" 
                    :preselect-first="false"
                    @input="isPropEmpty(arrayOfTypes[index])">
        </Multiselect>
        <Multiselect v-else
                    v-model="value[arrayOfTypes[index]]" :options="array" 
                    :multiple="true" 
                    track-by="someProperty"
                    label="someProperty"
                    :close-on-select="true" 
                    :clear-on-select="false" 
                    :preserve-search="true" 
                    placeholder="Выберите значение" 
                    :preselect-first="false"
                    @input="isPropEmpty(arrayOfTypes[index])">
        </Multiselect>
      </div>
    </template>
    </div>
      <SelectedValues :selected="value"/>
      <button class="state reload" @click="resetApp()">Перезагрузить</button>
      <br>
      <button v-for="(item, index) in stateArray" class="state" :class="{current :index == stateArray.length - 1, add: item.action == 'add', rem: item.action =='remove'}" :key="index" @click="rollbackState(index)">{{(item.action =="add" ? "+" : "-") + " " + item.element }}</button>
      <br>
      <button class="state cancel" v-show="stateArray.length > 1" @click="rollbackState(stateArray.length - 2)">Отмена</button>
  </div>
</template>

<script>
import Multiselect from 'vue-multiselect'
import SelectedValues from './SelectedValues.vue'

export default {
  name: 'TestTask',
  components: {
    Multiselect,
    SelectedValues
  },
  data() {
    return {
      loadMsg: '',
      fetchedData: [],
      flattenedArray: [],
      arrayOfTypes: [],
      typedArray: [],
      value: {},
      stateArray: [],
    }
  },
  created() {
    this.loadData('https://raw.githubusercontent.com/WilliamRu/TestAPI/master/db.json');
  },
  methods: {
    loadData (link) {
      let fetchStartTime = new Date().getTime();
      fetch(link)
      .then(async response => {
        this.loadMsg = "Загрузка...";
        const data = await response.json();
        this.fetchedData = data.testArr;
        this.arrayFlattening(this.fetchedData);
        this.findArrayTypes(this.flattenedArray);
        this.typizeArray(this.flattenedArray,this.arrayOfTypes);
        if(!response.ok) {
          const error = (data && data.message) || response.statusText;
          return Promise.reject(error);
        }
        let fetchEndTime = new Date().getTime() - fetchStartTime ;
        this.loadMsg = "Загружено за " + (fetchEndTime / 1000) + ' секунд'
      })
      .then(() => console.timeEnd)
      .catch(error => {
        this.loadMsg = error;
        console.error("Произошла ошибка!", error);
      });
    },
    isPropEmpty(propName) {
      if (this.value[propName].length == 0) delete this.value[propName]; 
    },
    arrayFlattening(arr) {
      for (let i = 0; i < arr.length; i++) {
        if(Array.isArray(arr[i])) {
          this.arrayFlattening(arr[i]);
        } else {
          this.flattenedArray.push(arr[i]);
        }
      }
    },
    findArrayTypes(arr) {
      for (let i = 0;i < arr.length; i++) {
        let elType = typeof(arr[i]);
        if (this.arrayOfTypes.indexOf(elType) < 0 ) {
          this.arrayOfTypes.push(elType);
        }
      }
      console.log("Типы массивов:" + this.arrayOfTypes);
        console.log("Всего типов массивов:" + this.arrayOfTypes.length);
    },
    typizeArray(arr, arrayOfTypes) {
      for (let i = 0; i < arrayOfTypes.length; i++) {
        this.typedArray.push([]);
      }
      for (let j = 0; j < arr.length; j++) {
        if (arr[j] !== null) {
            // Так как загружаемый объект пустой, это нужно расскоментировать для третьего селектора
            // if (typeof(arr[j]) == 'object') this.typedArray[arrayOfTypes.indexOf(typeof(arr[j]))].push({someProperty: 'test'},{someProperty: "test2"});
            this.typedArray[arrayOfTypes.indexOf(typeof(arr[j]))].push(arr[j]);
        }
      }
        
    },
    resetApp() {
      this.loadMsg= '';
      this.fetchedData= [];
      this.flattenedArray= [];
      this.arrayOfTypes= [];
      this.typedArray= [];
      this.value= {};
      setTimeout(()=> {
        this.stateArray = [];
      },10)
      this.loadData('https://raw.githubusercontent.com/WilliamRu/TestAPI/master/db.json');
    },
    addStateChange (action, group, element) {
      let changeObj = {
        action,
        group,
        element,
      }
      if (this.stateArray.length > 9) {
        this.stateArray.shift();
        this.stateArray.push(changeObj);
      } else {
        this.stateArray.push(changeObj);
      }
    },
    rollbackChange(changeObj) {
      if(changeObj.action == "add") {
        let group = this.value[changeObj.group];
        let elementIndex = group.indexOf(changeObj.element);
        console.log(group[elementIndex] + " удаляется")
        group.pop(elementIndex);
      } else {
        console.log(changeObj.element + " добавляется в " + this.value[changeObj.group]);
        this.value[changeObj.group].push(changeObj.element);
      }
    },
    rollbackState(stateIndex) {
      for (let i = this.stateArray.length - 1; i > stateIndex; i--) {
        this.rollbackChange(this.stateArray[i]);
        this.stateArray.pop(i);    
      }
    },
      
  },
  computed: {
    // watch the entire as a new object
    computedModel: function() {
      return Object.assign({}, this.value);
    }
  },
  watch: {
    computedModel: {
      handler: function(n, o) {
        let groupNameNew = Object.keys(n);
        let groupNameOld = Object.keys(o);
        if (groupNameNew.length < groupNameOld.length) {
          for (let index = 0; index < groupNameOld.length; index++) {
            if (!groupNameNew.includes(groupNameOld[index])) {
              let deletedGroup = groupNameOld[index];
              console.log("Удалена группа " + deletedGroup);
              console.log("Удален элемент " + o[deletedGroup][0]);

              this.addStateChange("remove", deletedGroup,o[deletedGroup][0]);
            }
          }
        } else if (groupNameNew.length > groupNameOld.length) {
          for (let index = 0; index < groupNameNew.length; index++) {
            if (!groupNameOld.includes(groupNameNew[index])) {
              let deletedGroup = groupNameNew[index];
              console.log("Добавлена группа " + deletedGroup);
              console.log("Добавлен элемент " + n[deletedGroup][0]);

              this.addStateChange("add", deletedGroup,n[deletedGroup][0]);

            }
          }
        } else {
          console.log("Группы не изменились");
          for (let i = 0; i < groupNameNew.length; i++) {
            let checkingGroupName = groupNameNew[i];
            if (n[checkingGroupName].length < o[checkingGroupName].length) {
              for (let j = 0; j < o[checkingGroupName].length; j++) {
                let checkingGroupElement = o[checkingGroupName][j];
                if (!n[checkingGroupName].includes(checkingGroupElement)) {
                  let deletedElement = checkingGroupElement;
                  console.log("Удален элемент " + deletedElement + " из группы " + checkingGroupName);

                  this.addStateChange("remove", checkingGroupName, deletedElement);

                 }
                
              }
            } else if (n[checkingGroupName].length > o[checkingGroupName].length) {
              for (let j = 0; j < n[checkingGroupName].length; j++) {
                let checkingGroupElement = n[checkingGroupName][j];
                if (!o[checkingGroupName].includes(checkingGroupElement)) {
                  let addedElement = checkingGroupElement;
                  console.log("Добавлен элемент " + addedElement + " в группу " + checkingGroupName);

                  this.addStateChange("add", checkingGroupName, addedElement);

                 }
                
              }
            } else {
              continue
            }
            
          }
        }
        
        console.log(groupNameNew, groupNameOld);
      },
      deep: true
    }
  },
}
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="sass">
@import 'TestTask' 
</style>
