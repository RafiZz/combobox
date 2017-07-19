<template>
  <div>
    <input type="text"
           v-model="search">
  
    <combo-box v-model="selectedOptions"
               :options="groups"
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
               :taggable="true"
               :multiple="true"
               :repeatable="false"
               placeholder="Выберите товар"
               width="300px"
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
               @blur="blur">
  
      <template slot="noResult"
                scope="props">
        <div class="options">{{ `Нет результатов по запросу "${props.text}"` }}</div>
      </template>
  
      <template slot="deleteIcon"
                scope="props">
        <span class="deleteIcon">×</span>
      </template>
  
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
      search: 'one',
      selectedOptions: [],
    };
  },
  methods: {
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

@keyframes wiggle {
  0%,
  100% {
    transform: rotate(55deg);
  }
  50% {
    transform: rotate(-50deg);
  }
}

.markedSymbol {
  display: inline-block;
  animation: wiggle 1.5s infinite;
  margin: 0 1px;
}
</style>
