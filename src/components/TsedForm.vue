<template>
    <div class="col col-5 mx-auto p-3 main">
    <div class="main-title"><h2>Price Components</h2></div>
    <b-row class="my-1 pl-2 text-right">
        <div 
            class="col col-21 my-2"
            id="total"
        >
            <h3>Total: {{ total | tsedFormat }} EUR/KG</h3>
        </div>
        <b-tooltip target="total" title="Full Total" placement="top" triggers="hover">
            {{total}}
        </b-tooltip>
    </b-row>
      <b-form  @submit.stop.prevent>
        <b-row v-for="(element, index) in tsedForm" 
            :key="index" 
            align-v="center" 
            class="my-1 pl-3" 
            :class="{'isSelected': element.hovered}"
            @mouseover="element.hovered = true"
            @mouseleave="element.hovered = false"
        >
            <div v-if="(element.hovered || element.hovered) && index > 0"
                class="icon erase clickable"
                @click="deletion(index)"
            >
                <b-icon 
                    icon="dash-circle-fill" 
                    variant="danger"
                >
                </b-icon>
            </div>

            <div class="col col-6 text-left">
                <template v-if="element.editMode">   <!-- label edit mode -->
                    <b-form-input
                        :ref="index+'-label'"
                        @blur="element.editMode = false"
                        @change="labelEdit(index, $event)"
                        type="text"
                        class="text-left pr-2"
                        :value="element.label"
                        :id="index.toString()+'-label'"
                        autocomplete="off"
                        >
                    </b-form-input>
                </template>
                <template v-else>   <!-- label view mode -->
                    <div
                        class="unbordered text-left ellipsis"
                        @mouseover="element.showEditIcon = true"
                        @mouseleave="element.showEditIcon = false"
                        :id="index+'-label'"
                    >
                        <div 
                            v-if="element.showEditIcon && index > 0"
                            class="clickable icon edit"
                            @click="switchToEdit(index, $event)"
                        >
                            <b-icon 
                                icon="pencil-fill" 
                                variant="success"
                            >
                            </b-icon>
                        </div>
                        {{element.label}}
                    </div>
                    <b-tooltip :target="index+'-label'" title="Full label" triggers="hover">
                        {{element.label}}
                    </b-tooltip>
                </template>

            </div>
            <div class="col col-6"> <!-- value field -->
                <b-form-input
                    :ref="index"
                    @focus="element.focusedValue = true"
                    @blur="element.editMode = false; element.focusedValue = false"
                    @change="element.editMode = false"
                    type="number"
                    class="text-right pr-2" 
                    :value="filterCall[index]"
                    @input="dataBack(index, $event)"
                    :id="index.toString()"
                    autocomplete="off"
                    @keyup="forbidden"
                >
                </b-form-input>
            </div>
            <b-tooltip :target="index.toString()" title="Full value" triggers="hover">
                {{element.value}}
            </b-tooltip>
        </b-row>

        <b-row align-v="center" class="mt-3 pl-3">
            <div class="col col-6 text-left">  <!-- ghost label -->
                <b-form-input 
                    class="ghost"
                    id="ghost-label"
                    for="ghost-value" 
                    placeholder="Label" 
                    autocomplete="off"
                    v-model="ghost.label" 
                    @change="ghostInput"
                >
                </b-form-input>
            </div>
            <div class="col col-6">  <!-- ghost input -->
                <b-form-input 
                    type="number" 
                    class="text-right pr-2 ghost" 
                    min="0" 
                    v-model="ghost.value" 
                    id="ghost-value" 
                    autocomplete="off"
                    placeholder="0.0"
                    @change="ghostInput" 
                    @keyup="forbidden"
                >
                </b-form-input>
            </div>
        </b-row>   
     </b-form>
    </div>
</template>

<script>
export default {
    data() {
      return {
          ghost : {
              label : '',
              value : '',
              hovered : false,
              focusedValue : false,
              showEditIcon : false,
              editMode : false
          },
          tsedForm : [
              {
                  label : 'Baseprice',
                  value : 1,
                  hovered : false,
                  focusedValue : false,
                  showEditIcon : false,
                  editMode : false
              }
          ]
      }
    },
    filters : {
        tsedFormat(value) {
            if (!value) {
                return "0.0";
            }
            
            const decimals = value % 1 ?value.toString().split(".")[1].length : 0;

            value = Number(value);

            if (decimals > 1) {
                return value.toFixed(2);
            }
            if (decimals < 2) {
                return value.toFixed(1)
            }

        }
    },
    computed: {
      total() {
        return this.tsedForm.map(a => a.value).reduce((a, b) => Number(a) + Number(b), 0);
      },

      validation() {
        return this.userId.length > 4 && this.userId.length < 13
      },

      filterCall() {
        let filterSet = [];
        let vm = this;

        this.tsedForm.forEach(function (item) {
            if (item.focusedValue) {
                filterSet.push(item.value);
            } else {
                filterSet.push(vm.$options.filters.tsedFormat(item.value));
            }
        });

        return filterSet;
      }

    },
    methods : {
        labelEdit(target, event) {
            if (event) {
                this.tsedForm[target].label = event;
            }
            this.tsedForm[target].editMode = false;
        },

        switchToEdit(index) {
            let vm = this;
            this.tsedForm[index].editMode = true;

            this.$nextTick(() => {
                vm.$refs[index+'-label'][0].focus();
            });
        },

        deletion(target) {
            this.tsedForm.splice(target, 1);
        },

        dataBack(target, event) {
            this.tsedForm[target].value = event;
        },

        ghostError(target) {
            const domElement = document.querySelector(target);
            domElement.classList.add('error-sign');
            setTimeout(() => {
                domElement.classList.remove('error-sign');
            }, 1000);
        },

        ghostInput() {
            this.ghost.label = this.ghost.label.trim();
            const labelCheck = ( this.ghost.label.match(/[^0-9]/) && this.ghost.label.length > 1 ) ? true : false;
            let decimalCheck = false;
            if (!labelCheck) {
                this.ghostError('#ghost-label');
            }

            if (this.ghost.value.length > 1) {
               if (/^-?\d+(?:[.,]\d*?)$/.test(this.ghost.value)) decimalCheck = true;
            }

            if (!decimalCheck) {
                this.ghostError('#ghost-value');
            }
           
            if (!labelCheck || !decimalCheck) {
                return;
            }
          
                const newElement = {
                   ...this.ghost
                };

                this.tsedForm.push(newElement);
                this.ghost.label = '';
                this.ghost.value = '';
    
        },

        forbidden(event) {
            if(event.key == "-") {
                event.target.value = event.target.value.slice(0, -1);
            }
        }
    },

    created() {
    document.addEventListener('focusin', this.focused)
    },

    beforeDestroy() {
    document.removeEventListener('focusin', this.focused)
    },
}
</script>

<style scoped>
.main {
    background-color : white;
    border : 5px solid black;
    border-radius: 10px;
}
.main-title {
    position: absolute;
    top: -23px;
    background-color: white;
    padding: 0 5px 0 5px;
}
.main-title h2 {
    padding : 0;
    margin : 0;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type=number] {
  -moz-appearance: textfield;
  border : 1px solid black;
}

.ghost {
    color : #a1a1a1!important;
    border : 1px solid #a1a1a1!important;
}
::placeholder {
    color : #a1a1a1!important;
}

.error-sign {
    border : 1px solid red!important;
}

label {
    margin-bottom: 0;
}

.isSelected {
    background-color : #fffbbd;
}

.icon {
    position : absolute;
    background : transparent;
}
.erase {
    left : 6px;
}

.clickable {
    cursor : pointer!important;
    z-index: 100;
}

.unbordered {
    width : 100%;
    height: 37.9972px;
    display : flex;
    flex-direction: row;
    align-content: center;
    justify-content: flex-start;
    padding : 6px 8px 6px 12px;
    margin-top: 7px;
}

.edit {
    right : 0px;
}

.ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    display: inline-block;
    white-space: nowrap;
}

</style>