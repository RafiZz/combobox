<template>
  <div>
    <input type="text"
           v-model="search">
  
    <combo-box v-model="selectedOptions"
               :options="options"
               groupLabel="name"
               groupValues="values"
               optionLabel="name"
               optionValue="id"
               :initOpen="true"
               :closeOnSelect="false"
               :closeOnBlur="true"
               :hideSelected="true"
               :resetAfter="false"
               :searchable="true"
               :multiple="true"
               :repeatable="false"
               placeholder="Выберите товар"
               width="400px"
               :maxHeight="200"
               :transitionSpeed="{ enter: 1000, leave: 1500 }"
               position="bottom"
               :styleSelectOptionInDropdown="{ color: 'blue' }"
               :caseSensitivity="false"
               :searchValue="search"
               @searchChange="searchChange"
               @open="open"
               @close="close"
               @select="select"
               @focusEl="focusEl"
               @blurEl="blurEl"
               @focus="focus"
               @blur="blur"
               :required="true"
               url="https://jsonplaceholder.typicode.com/posts"
               :pageSize="15"
               :debounce="2000"
               :successCallback="successCallback"
               :errorCallback="errorCallback"
               :showNoResult="false"
               :showFirstOption="false"
               :showLastOption="false"
               :internalSearch="false"
               :tag="tag">
  
      <template slot="noResult"
                scope="props">
        <div class="options">{{ `Нет результатов по запросу "${props.text}"` }}</div>
      </template>
  
      <template slot="deleteIcon"
                scope="props">
        <span class="deleteIcon">×</span>
      </template>
  
      <div slot="loading"
           class="spinner">
        <div class="rect1"></div>
        <div class="rect2"></div>
        <div class="rect3"></div>
        <div class="rect4"></div>
        <div class="rect5"></div>
      </div>
  
      <template slot="resetButton"
                scope="props">
        <span class="deleteIcon">×</span>
      </template>
  
      <template slot="markedSymbol"
                scope="props">
        <b class="markedSymbol">{{ props.text }}</b>
      </template>
  
      <template slot="groupLabel"
                scope="props">
        <div style="border-top: 1px solid black; text-align: center;">{{ props.text }}</div>
      </template>
  
      <template slot="firstOption"
                scope="props">
        <div class="options">{{ `Создать ${props.text ? '"' + props.text + '"' : 'новый товар'}` }}</div>
      </template>
  
      <template slot="allOptions"
                scope="props">
        <div class="options"
             style="color: red">+ {{ props.text }} +</div>
      </template>
  
      <template slot="selectedOptions"
                scope="props">
        <div class="options">- {{ props.text }} -</div>
      </template>
  
      <template slot="lastOption"
                scope="props">
        <div class="options">{{ `Создать ${props.text ? '"' + props.text + '"' : 'новый товар'}` }}</div>
      </template>
  
    </combo-box>
  
  </div>
</template>

<script>
import ComboBox from './ComboBox'

export default {
  name: 'hello',
  data() {
    return {
      search: '',
      selectedOptions: [],
      options: [],
    };
  },
  mounted() {
    this.options = this.groups;
  },
  methods: {
    tag(searchText) {
      global.console.log('tag', searchText);
      this.selectedOptions.push({ id: searchText, name: `${searchText} from parrent` });
    },
    successCallback(data, page) {
      // const options = data;
      const options = data
        ? data.map(object => ({ id: object.id, name: object.title }))
        : [];
      const group = {
        name: `From Api ${page}`,
        values: options,
      };
      if (page === 1) {
        this.options = [group];
      } else {
        this.options.push(group);
      }
    },
    errorCallback(error) {
      global.window.alert('parentErrorCallback', error.status);
    },
    searchChange(newValue, oldValue) {
      global.console.log('searchChange', `${oldValue} => ${newValue}`);
    },
    open() {
      global.console.log('open');
    },
    close() {
      global.console.log('close');
    },
    select(option) {
      global.console.log('select', option);
    },
    focusEl(value) {
      global.console.log('focusEl', value);
    },
    blurEl(value) {
      global.console.log('blurEl', value);
    },
    focus() {
      global.console.log('focus component');
    },
    blur() {
      global.console.log('blur component');
    },
  },
  computed: {
    strings() {
      return ['Zero', 'One', 'Two', 'Three', 'Four', 'Five', 'Six', 'Seven', 'Eight', 'Nine', 'Ten', 'Eleven', 'Twelve', 'Thirteen', 'Fourteen', 'Fifteen', 'Sixteen', 'Seventeen', 'Eighteen', 'Nineteen', 'Twenty', 'Twenty-one', 'Twenty-two', 'Twenty-three', 'Twenty-four', 'Twenty-five', 'Twenty-six', 'Twenty-seven', 'Twenty-eight', 'Twenty-nine', 'Thirty', 'Thirty-one', 'Thirty-two', 'Thirty-three', 'Thirty-four', 'Thirty-five', 'Thirty-six', 'Thirty-seven', 'Thirty-eight', 'Thirty-nine'];
    },
    objects() {
      return this.strings.map((number, index) => ({ id: index, name: number }));
    },
    groups() {
      return [
        { name: 'Group-1', values: this.objects.slice(0, 10) },
        { name: 'Group-2', values: this.objects.slice(10, 20) },
        { name: 'Group-3', values: this.objects.slice(20, 30) },
        { name: 'Group-4', values: this.objects.slice(30, 40) },
      ];
    },
  },
  components: {
    ComboBox,
  }
};
</script>

<style scoped>
div.main {
  display: flex;
  align-items: flex-start;
}

.options-all {
  padding: 3px;
}

.deleteIcon {
  color: red;
}

.markedSymbol {
  display: inline-block;
  animation: wiggle 1.5s infinite;
  margin: 0 1px;
  color: black;
}

@keyframes wiggle {
  0%,
  100% {
    transform: rotate(55deg);
  }
  50% {
    transform: rotate(-50deg);
  }
}

.spinner {
  margin: 100px auto;
  width: 50px;
  height: 40px;
  text-align: center;
  font-size: 10px;
}

.spinner>div {
  background-color: #333;
  height: 100%;
  width: 6px;
  display: inline-block;

  -webkit-animation: sk-stretchdelay 1.2s infinite ease-in-out;
  animation: sk-stretchdelay 1.2s infinite ease-in-out;
}

.spinner .rect2 {
  -webkit-animation-delay: -1.1s;
  animation-delay: -1.1s;
}

.spinner .rect3 {
  -webkit-animation-delay: -1.0s;
  animation-delay: -1.0s;
}

.spinner .rect4 {
  -webkit-animation-delay: -0.9s;
  animation-delay: -0.9s;
}

.spinner .rect5 {
  -webkit-animation-delay: -0.8s;
  animation-delay: -0.8s;
}

@-webkit-keyframes sk-stretchdelay {
  0%,
  40%,
  100% {
    -webkit-transform: scaleY(0.4)
  }
  20% {
    -webkit-transform: scaleY(1.0)
  }
}

@keyframes sk-stretchdelay {
  0%,
  40%,
  100% {
    transform: scaleY(0.4);
    -webkit-transform: scaleY(0.4);
  }
  20% {
    transform: scaleY(1.0);
    -webkit-transform: scaleY(1.0);
  }
}
</style>
