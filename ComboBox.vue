<template>
  <div ref="combobox"
       @mousedown.prevent.stop="toggleDropdown"
       class="combobox"
       :style="{'width': width}">
  
    <div class="display">
      <template v-if="multiple && !resetAfter">
        <div class="tag"
             v-for="(option, i) in selectedOptions"
             :key="i">
          <span>{{ getOptionLabel(option) }}</span>
          <span @mousedown.prevent.stop="deselect(i)"
                class="delete-button">
            <slot name="deleteIcon"></slot>
            <span v-if="!$scopedSlots.deleteIcon">×</span>
          </span>
        </div>
      </template>
  
      <input type="search"
             ref="search"
             v-model="search"
             @keydown.up.prevent="changeTypeAheadPointer('up')"
             @keydown.down.prevent="changeTypeAheadPointer('down')"
             @keydown.enter.prevent="typeAheadEnter"
             @keydown.delete="typeAheadDeselect"
             @keydown.esc="typeAheadEsc"
             @blur="blur"
             @focus="focus"
             :placeholder="currentPlaceholder"
             :readonly="readonly">
  
      <template v-if="!search && !multiple && selectedOptions.length">
        <span @mousedown.prevent.stop="deselect()"
              class="delete-button">
          <slot name="resetButton"></slot>
          <span v-if="!$scopedSlots.resetButton">×</span>
        </span>
      </template>
  
    </div>
    <ul class="dropdown"
        ref="dropdown"
        @mousedown.prevent.stop
        :style="dropdownStyles">
  
      <li v-if="$scopedSlots.firstOption"
          :class="{ highlight: typeAheadPointer === 'first' }"
          @mouseover="typeAheadPointer = 'first'"
          @mousedown.prevent.stop="tag">
        <slot name="firstOption"
              :text="search"></slot>
      </li>
  
      <li v-if="noResult"
          :class="{ highlight: typeAheadPointer === 'noResult' }"
          @mouseover="typeAheadPointer = 'noResult'"
          @mousedown.prevent.stop="tag">
        <slot name="noResult"
              :text="search"></slot>
        <span v-if="$scopedSlots.noResult">No result.</span>
      </li>
  
      <!-- for groups -->
      <template v-else-if="isGroups()">
        <li v-for="(group, x) in allOptions"
            v-if="showGroup(group)"
            :key="x">
          <slot name="groupLabel"
                :text="getGroupLabel(group)"></slot>
          <template v-if="!$scopedSlots.groupLabel && getGroupLabel(group)">{{ getGroupLabel(group) }}</template>
  
          <ul>
            <li v-for="(option, y) in getGroupValues(group)"
                v-if="showOption(option)"
                :key="y"
                :class="{ highlight: typeAheadPointer === `${x}.${y}` }"
                @mouseover="typeAheadPointer = `${x}.${y}`"
                @mousedown.prevent.stop="select(`${x}.${y}`)"
                :style="isSelected(option) && styleSelectOptionInDropdown">
  
              <marked-text :originalSlot="isSelected(option) ? $scopedSlots.selectedOptions : $scopedSlots.allOptions"
                           :markedSymbolSlot="$scopedSlots.markedSymbol"
                           :text="getOptionLabel(option)"
                           :searchText="search"
                           :sensitivity="caseSensitivity"></marked-text>
            </li>
          </ul>
        </li>
      </template>
  
      <!-- for objects or strings -->
      <template v-else>
        <li v-if="showOption(option)"
            v-for="(option, x) in allOptions"
            :key="x"
            :class="{ highlight: typeAheadPointer === x }"
            @mouseover="typeAheadPointer = x"
            @mousedown.prevent.stop="select(x)"
            :style="isSelected(option) && styleSelectOptionInDropdown">
          :text="getOptionLabel(option)"></slot> -->
          <marked-text :originalSlot="isSelected(option) ? $scopedSlots.selectedOptions : $scopedSlots.allOptions"
                       :markedSymbolSlot="$scopedSlots.markedSymbol"
                       :text="getOptionLabel(option)"
                       :searchText="search"
                       :sensitivity="caseSensitivity"></marked-text>
        </li>
      </template>
  
      <li v-if="$scopedSlots.lastOption"
          :class="{ highlight: typeAheadPointer === 'last' }"
          @mouseover="typeAheadPointer = 'last'"
          @mousedown.prevent.stop="tag">
        <slot name="lastOption"
              :text="search"></slot>
      </li>
    </ul>
  
  </div>
</template>

<script>
const markedText = {
  render(createElement) {
    const el = this.originalSlot ? this.originalSlot({ text: this.text })[0] : createElement('div', this.text);
    const { tag, data } = el;
    const innerHtml = el.children[0].text;
    const [first, last] = innerHtml.split(this.text);
    const children = () => {
      const array = [];
      const index = this.sensitivity ? this.text.indexOf(this.searchText) : this.text.toLowerCase().indexOf(this.searchText.toLowerCase());
      if (index > -1) {
        const start = first + this.text.slice(0, index);
        const middle = this.text.substr(index, this.searchText.length);
        const end = this.text.substr(index + this.searchText.length) + last;
        if (start) array.push(start);
        for (let i = 0; i < middle.length; i += 1) {
          array.push(this.markedSymbolSlot ? this.markedSymbolSlot({ text: middle[i] }) : createElement('b', middle[i]));
        }
        if (end) array.push(end);
      } else {
        array.push(this.text);
      }
      return array;
    }
    return createElement(
      tag,
      data,
      children()
    )
  },
  props: {
    text: {
      type: String,
      required: true,
    },
    searchText: {
      type: String,
      required: true,
    },
    originalSlot: {
      type: Function,
      required: false,
    },
    markedSymbolSlot: {
      type: Function,
      required: false,
    },
    sensitivity: {
      type: Boolean,
      required: false,
    },
  },
};

export default {
  name: 'ComboBox',
  model: {
    prop: 'selectedOptions',
    event: 'update:selectedOptions',
  },
  components: { markedText },
  props: {
    options: {
      type: Array,
      required: true,
    },
    selectedOptions: {
      type: Array,
      default: []
    },
    optionLabel: {
      type: String,
      default: ''
    },
    optionValue: {
      type: String,
      default: ''
    },
    groupLabel: {
      type: String,
      default: ''
    },
    groupValues: {
      type: String,
      default: ''
    },
    width: {
      type: String,
      default: '200px'
    },
    maxHeight: {
      type: Number,
      default: 200
    },
    position: {
      type: String,
      default: 'bottom'
    },
    transitionSpeed: {
      type: [Number, Object],
      default: 1000
    },
    placeholder: {
      type: String,
      default: ''
    },
    closeOnSelect: {
      type: Boolean,
      default: false
    },
    closeOnBlur: {
      type: Boolean,
      default: false
    },
    hideSelected: {
      type: Boolean,
      default: false
    },
    resetAfter: {
      type: Boolean,
      default: false
    },
    taggable: {
      type: Boolean,
      default: true
    },
    multiple: {
      type: Boolean,
      default: true
    },
    caseSensitivity: {
      type: Boolean,
      default: false
    },
    repeatable: {
      type: Boolean,
      default: false
    },
    searchable: {
      type: Boolean,
      default: true
    },
    searchValue: {
      type: String,
      default: ''
    },
    styleSelectOptionInDropdown: {
      type: Object,
      default: { 'text-decoration': 'line-through' }
    },
    initOpen: {
      type: Boolean,
      default: false
    },
  },
  data() {
    return {
      search: this.searchable ? this.searchValue : '',
      readonly: !this.searchable,
      open: this.initOpen,
      newOptions: [],
      currentPosition: this.position,
      currentSelectedOptions: this.selectedOptions,
      typeAheadPointer: '',
      height: this.initOpen ? this.maxHeight : 0,
    }
  },
  computed: {
    currentPlaceholder() {
      let placeholder = this.placeholder;
      if (this.currentSelectedOptions.length && !this.multiple) {
        placeholder = '';
        for (let i = 0; i < this.currentSelectedOptions.length; i += 1) {
          placeholder += `${this.getOptionLabel(this.currentSelectedOptions[i])}, `;
        }
        placeholder = placeholder.slice(0, placeholder.length - 2);
      }
      return placeholder;
    },
    allOptions() {
      let newOptions = [];
      if (!this.isGroups()) {
        newOptions = this.newOptions;
      } else if (this.newOptions && this.newOptions.length) {
        const newGroup = {};
        newGroup[this.groupLabel] = 'New group';
        newGroup[this.groupValues] = this.newOptions;
        newOptions = [newGroup];
      }
      return this.options.concat(newOptions);
    },
    typeAheadPointerValues() {
      const values = [];

      if (!this.isGroups()) {
        this.allOptions.forEach((option, index) => this.showOption(option) && values.push(index));
      } else {
        this.allOptions.forEach((group, index) =>
          this.getGroupValues(group).forEach((option, innerIndex) =>
            this.showOption(option) && values.push(`${index}.${innerIndex}`)
          )
        )
      }

      if (!values.length) {
        values.push('noResult');
      }

      if (this.$scopedSlots.firstOption) values.unshift('first');
      if (this.$scopedSlots.lastOption) values.push('last');

      return values;
    },
    noResult() {
      const noResultPointerIndex = this.$scopedSlots.firstOption ? 1 : 0;
      const pointer = this.typeAheadPointerValues && this.typeAheadPointerValues[noResultPointerIndex];
      if (pointer === 'noResult') {
        this.typeAheadPointer = this.typeAheadPointerValues[noResultPointerIndex];
      }
      return pointer === 'noResult';
    },
    transitionDuration() {
      let ms;
      if (typeof this.transitionSpeed === 'object') {
        ms = this.open ? this.transitionSpeed.enter : this.transitionSpeed.leave;
      }
      return ms || this.transitionSpeed;
    },
    dropdownStyles() {
      const styles = {
        'max-height': `${this.open ? this.height : 0}px`,
        'width': this.width,
        'transition-property': 'max-height',
        'transition-duration': `${this.transitionDuration}ms`,
      };
      if (this.currentPosition === 'top') {
        styles.bottom = '42px';
        if (this.$refs.display) {
          styles.bottom = `${this.$refs.display.clientHeight}px`;
        }
      }
      if (!this.open) {
        styles.borderTop = 'none';
        styles.borderBottom = 'none';
        styles.boxShadow = 'none';
      }
      return styles;
    },
  },
  watch: {
    searchValue() {
      this.search = this.searchable ? this.searchValue : '';
    },
    search(newValue, oldValue) {
      if (this.searchable) {
        this.typeAheadPointer = this.typeAheadPointerValues[this.$scopedSlots.firstOption ? 1 : 0];
      }
      if (newValue.length > oldValue.length) {
        this.open = true;
      }
      this.$emit('searchChange', newValue, oldValue);
    },
    currentSelectedOptions(newValue) {
      this.$emit('update:selectedOptions', newValue);
    },
    selectedOptions(newValue) {
      this.currentSelectedOptions = newValue;
    },
    open(newValue) {
      this.$emit(newValue ? 'open' : 'close');
    },
    typeAheadPointer(newValue, oldValue) {
      const forTagPointers = [undefined, 'first', 'noResult', 'last', ''];

      if (forTagPointers.indexOf(oldValue) === -1) {
        this.$emit('blurEl', this.getOptionByPointer(oldValue));
      }

      if (forTagPointers.indexOf(newValue) === -1) {
        this.$emit('focusEl', this.getOptionByPointer(newValue));
      }
    },
  },
  methods: {
    getOptionByPointer(pointer) {
      const pointers = [undefined, 'first', 'noResult', 'last', ''];
      let option;
      if (pointers.indexOf(pointer) === -1) {
        if (this.isGroups()) {
          const [x, y] = pointer.toString().split('.');
          option = this.getGroupValues(this.allOptions[x])[y];
        } else {
          option = this.allOptions[pointer];
        }
      }
      return option;
    },
    isGroups() {
      return this.groupLabel && this.groupValues;
    },
    isObjects() {
      return this.optionLabel && this.optionValue;
    },
    getOptionLabel(option) {
      return this.isObjects() ? option[this.optionLabel] : option;
    },
    getOptionValue(option) {
      return this.isObjects() ? option[this.optionValue] : option;
    },
    getGroupLabel(group) {
      return group && Object.prototype.hasOwnProperty.call(group, this.groupLabel)
        && group[this.groupLabel];
    },
    getGroupValues(group) {
      return group && Object.prototype.hasOwnProperty.call(group, this.groupValues)
        && group[this.groupValues];
    },
    includeSearchString(string) {
      if (this.caseSensitivity) {
        return string.indexOf(this.search) > -1;
      }
      return string.toLowerCase().indexOf(this.search.toLowerCase()) > -1;
    },
    showOption(option) {
      let show = true;

      if (this.hideSelected && !this.readonly) {
        show = !this.selectedOptions.some(selectedOption => this.getOptionValue(selectedOption) === this.getOptionValue(option))
          && this.includeSearchString(this.getOptionLabel(option));
      } else if (!this.hideSelected && !this.readonly) {
        show = this.includeSearchString(this.getOptionLabel(option));
      } else if (this.hideSelected && this.readonly) {
        show = !this.selectedOptions.some(selectedOption => this.getOptionValue(selectedOption) === this.getOptionValue(option));
      }

      return show;
    },
    showGroup(group) {
      return group && this.getGroupValues(group) && this.getGroupValues(group).some(option => this.showOption(option));
    },
    isSelected(option) {
      return option && this.selectedOptions &&
        this.selectedOptions.some(selectedOption => this.getOptionValue(selectedOption) === this.getOptionValue(option));
    },
    toggleDropdown() {
      if (this.open) {
        this.$refs.search.focus();
      } else {
        this.computePosition();
      }
      this.open = !this.open;
    },
    computePosition() {
      const combobox = this.$refs.combobox;
      const dropdown = this.$refs.dropdown;
      if (!combobox || !dropdown) return;
      const clientHeight = document.documentElement.clientHeight;
      let dropdownNeededHeight = 0;
      for (let i = 0; i < dropdown.children.length; i += 1) {
        dropdownNeededHeight += dropdown.children[i].offsetHeight;
      }
      const distanceTo = combobox.getBoundingClientRect();
      const distanceToTop = distanceTo.top;
      const distanceToBottom = clientHeight - distanceTo.bottom;
      const isFitBottom = this.maxHeight < distanceToBottom;
      const isFitTop = this.maxHeight < distanceToTop;

      if (!isFitBottom && distanceToTop > distanceToBottom) {
        this.currentPosition = 'top';
        this.height = Math.min(this.maxHeight, distanceToTop, dropdownNeededHeight);
      } else if (!isFitTop && distanceToBottom > distanceToTop) {
        this.currentPosition = 'bottom';
        this.height = Math.min(this.maxHeight, distanceToBottom, dropdownNeededHeight);
      } else {
        this.currentPosition = this.position;
        this.height = this.maxHeight;
      }
    },
    scrollTo(type) {
      const el = this.$refs.dropdown.querySelector("li[class='highlight']")
      if (el) el.scrollIntoView(type === 'down');
    },
    changeTypeAheadPointer(type) {
      if (!this.typeAheadPointer && this.typeAheadPointer !== 0) {
        this.typeAheadPointer = this.typeAheadPointerValues[this.$scopedSlots.firstOption ? 1 : 0];
        return;
      }

      let pointer;
      const currentPointerIndex = this.typeAheadPointerValues.indexOf(this.typeAheadPointer);

      if (type === 'down') {
        pointer = currentPointerIndex + 1 < this.typeAheadPointerValues.length
          ? this.typeAheadPointerValues[currentPointerIndex + 1]
          : this.typeAheadPointerValues[this.typeAheadPointerValues.length - 1];
      } else if (type === 'up') {
        pointer = currentPointerIndex - 1 >= 0
          ? this.typeAheadPointerValues[currentPointerIndex - 1]
          : this.typeAheadPointerValues[0];
      }

      this.typeAheadPointer = pointer;

      this.scrollTo(type);
    },
    select(pointer) {
      const option = this.getOptionByPointer(pointer || this.typeAheadPointer);
      let canBeSelected = true;

      if (!this.repeatable) {
        const index = this.currentSelectedOptions.indexOf(option);
        if (index > -1) {
          canBeSelected = false;
          this.deselect(index);
        }
      }
      if (canBeSelected) {
        this.$emit('select', option);
        this.typeAheadPointer = '';
        if (this.multiple) {
          this.currentSelectedOptions.push(option);
          this.search = '';
          this.readonly = !this.searchable;
        } else {
          this.currentSelectedOptions = [option];
        }
      }
    },
    deselect(index) {
      this.currentSelectedOptions.splice(index, 1);
    },
    tag() {
      if (!this.searchable || this.readonly || !this.search) {
        this.readonly = false;
        this.$refs.search.focus();
        return;
      }
      this.readonly = !this.searchable;
      let option = this.search;
      if (this.isObjects()) {
        option = {};
        option[this.optionLabel] = this.search;
        option[this.optionValue] = this.search;
      }
      if (this.repeatable || (!this.newOptions.indexOf(option) > -1)) {
        this.newOptions.push(option);
        if (this.multiple) {
          this.currentSelectedOptions.push(option)
        } else {
          this.currentSelectedOptions = [option];
        }
      }
      this.search = '';
    },
    typeAheadEnter() {
      const forTagPointers = ['first', 'noResult', 'last'];
      const pointer = this.typeAheadPointer && this.typeAheadPointer !== 0
        ? this.typeAheadPointer
        : this.typeAheadPointerValues[this.$scopedSlots.firstOption ? 1 : 0];
      if (forTagPointers.indexOf(pointer) === -1) {
        this.select(pointer);
      } else {
        this.tag();
      }
    },
    typeAheadDeselect() {
      global.console.log(!this.search.length)
      global.console.log(this.currentSelectedOptions.length)
      if (!this.search.length && this.currentSelectedOptions.length) {
        this.deselect(this.currentSelectedOptions.length - 1);
      }
    },
    typeAheadEsc() {
      if (!this.search.length) {
        this.open = false;
      }
    },
    focus() {
      this.$emit('focus');
    },
    blur() {
      if (this.closeOnBlur) {
        this.open = false;
      }
      this.$emit('blur');
    },
  },
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.combobox {
  display: inline-block;
  min-height: 40px;
  position: relative;
}
.combobox .display {
  display: flex;
  flex-wrap: wrap;
  border: 1px solid black;
  height: 100%;
  width: 100%;
  cursor: text;
}
.combobox .display .tag {
  flex-grow: 1;
  position: relative;
  color: #ffffff;
  background-color: #41B783;
  border: 1px solid #ccc;
  border-radius: 4px;
  height: 26px;
  margin: 4px 1px 0px 3px;
  padding: 1px 0.25em;
  padding-right: 20px;
  line-height: 24px;
  max-width: 100%;
}
.combobox .display .tag .delete-button {
  position: absolute;
  top: 0;
  bottom: 0;
  right: 5px;
  display: flex;
  align-items: center;
  cursor: pointer;
}
.combobox .display input[type="search"] {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  line-height: 1.42857143;
  font-size: 1em;
  height: 34px;
  flex-grow: 1;
  border: none;
  outline: none;
  margin: 0;
  padding: 0 .5em;
  background: none;
  box-shadow: none;
  clear: none;
}
.combobox .display .delete-button {
  margin-right: 10px;
  display: flex;
  align-items: center;
  cursor: pointer;
}
.combobox ul {
  list-style: none;
}
.combobox ul.dropdown {
  position: absolute;
  overflow: auto;
  background-color: rgba(255, 255, 255, 0.95);
  border-radius: 3px;
  z-index: 1000;
  border: 1px solid rgba(0, 0, 0, 0.26);
  box-shadow: 0px 3px 6px 0px rgba(0, 0, 0, 0.15);
  border-radius: 0 0 4px 4px;
  text-align: left;
}
.combobox ul li {
  cursor: pointer;
  padding: 2px;
}
.combobox ul li.highlight {
  background: #41B783;
  color: #fff;
}
</style>
