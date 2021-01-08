<template>
  <div class="col-lg-4 offset-lg-4">
    <h2 class="border-bottom mb-4 pb-2">Frankies Treeview</h2>
    <button
      v-on:click="emit"
      :disabled="enabledSaveBtn"
      class="btn btn-primary"
    >
      Speichern
    </button>
    <div class="border-bottom mb-4 mt-4"></div>
    <div class="border-bottom">
      <ul id="tree" class="mt-3 mb-0">
        <li v-for="(parent, parentidx) in treeData" :key="parentidx">
          <span class="d-inline-block">
            <span
              :class="[iconClass, parent.visibleChild ? openClass : '']"
              v-on:click="parent.visibleChild = !parent.visibleChild"
            ></span>
            <span class="custom-control custom-checkbox d-inline-block">
              <input
                type="checkbox"
                :indeterminate.prop="parent.indeterminate"
                class="custom-control-input"
                :id="'parent_' + parentidx"
                :checked="parent.checked"
                @click="
                  changeParentChkbx(parent.checked, parent.children, parentidx)
                "
              />
              <label
                class="custom-control-label"
                :for="'parent_' + parentidx"
                >{{ parent.name }}</label
              >
            </span>
          </span>
          <ul v-show="parent.visibleChild">
            <li v-for="(child, childIdx) in parent.children" :key="childIdx">
              <span class="custom-control custom-checkbox d-inline-block">
                <input
                  type="checkbox"
                  class="custom-control-input"
                  :id="'child_' + parentidx + childIdx"
                  :checked="child.checked"
                  @click="
                    changeChildChkbx(
                      child.checked,
                      parent.children,
                      parentidx,
                      childIdx
                    )
                  "
                />
                <label
                  class="custom-control-label"
                  :for="'child_' + parentidx + childIdx"
                  >{{ child.name }}</label
                >
              </span>
              <span class="d-inline-block label"></span>
            </li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "FraenkiesTreeview",
  props: {
    myData: {
      type: Array
    },
    showbyChecked: {
      type: Boolean,
      default: false
    },
    disablingSaveBtn: {
      typ: Boolean,
      default: true
    }
  },
  data() {
    return {
      treeData: [],
      iconClass: "toggle-icon",
      openClass: "open",
      selectedValues: []
    };
  },
  computed: {
    enabledSaveBtn() {
      if (this.disablingSaveBtn === true) {
        if (this.selectedValues.length > 0) {
          return false;
        } else {
          return true;
        }
      } else {
        return false;
      }
    }
  },
  created() {
    this.myData.forEach((parentItem) => {
      this.treeData.push({
        ...parentItem,
        visibleChild: false,
        indeterminate: false,
        checked: false
      });
    });

    this.treeData.forEach((parentItem) => {
      parentItem.children.forEach((childItem) => {
        childItem.checked = false;
      });
    });
  },
  methods: {
    emit: function () {
      this.$emit("event-child", this.selectedValues);
    },
    changeParentChkbx(parentChecked, children, parentIdx) {
      var parentNode = this.treeData[parentIdx];
      parentNode.checked = !parentChecked;

      parentNode.indeterminate = false;

      // show/hide child-items by checked/unchecked parent-item
      if (this.showbyChecked === true) {
        parentNode.visibleChild = parentNode.checked;
      }

      // checked all child checkboxes
      for (let index = 0; index < parentNode.children.length; index++) {
        parentNode.children[index].checked = parentNode.checked;
      }

      this.getSelectedChkbx();
    },
    changeChildChkbx(childrenChecked, children, parentIdx, childIdx) {
      var parentNode = this.treeData[parentIdx];
      parentNode.children[childIdx].checked = !childrenChecked;

      // set indeterminate
      var counter = 0;
      for (let i = 0; i < children.length; i++) {
        if (children[i].checked) {
          counter++;
        }
      }

      // parentNode.indeterminate = !parentNode.indeterminate;

      if (counter === 0) {
        parentNode.indeterminate = false;
        parentNode.checked = false;
      } else {
        if (counter < children.length) {
          parentNode.indeterminate = true;
          parentNode.checked = false;
        } else {
          parentNode.indeterminate = false;
          parentNode.checked = true;
        }
      }

      this.getSelectedChkbx();
    },
    getSelectedChkbx() {
      const parentNode = this.treeData;
      this.selectedValues = [];

      for (let idx = 0; idx < parentNode.length; idx++) {
        var childNode = parentNode[idx].children;

        for (let idx2 = 0; idx2 < childNode.length; idx2++) {
          if (childNode[idx2].checked) {
            this.selectedValues.push(childNode[idx2].name);
          }
        }
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
ul#tree {
  list-style: none;
  padding-left: 0;
  text-align: left;

  > li {
    margin-bottom: 15px;

    .toggle-icon {
      position: absolute;
      margin-top: 4px;
      display: block;
      width: 24px;
      height: 24px;
      cursor: pointer;
    }

    .toggle-icon::before {
      position: absolute;
      font-family: "Material Design Icons";
      content: "\F0415";
      font-size: 21px;
      top: -5px;
      margin-left: -1px;
      color: #90A4AE; // md Blue Grey 300
      left: 1px;
    }

    .toggle-icon.open::before {
      content: "\F0374";
    }

    span.custom-control {
      font-weight: normal;
      font-size: 21px;
      margin-left: 40px;

      .custom-control-input:checked ~ .custom-control-label::before {
        color: #fff;
        border-color: #00B8D4; // md Cyan A700
        background-color: #00B8D4;
      }

      .custom-control-input:indeterminate ~ .custom-control-label::before {
        border-color: #00B8D4; // md Cyan A700
        background-color: #00B8D4;
      }
    }
    > ul {
      list-style: none;
      margin-left: 1px;
      > li {
        margin-left: 25px;
      }
    }
  }

  .custom-control-label {
    &::before,
    &::after {
      top: 0.45rem;
    }
  }
}
</style>
