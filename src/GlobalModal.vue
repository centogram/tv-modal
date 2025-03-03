<template>
  <ModalBase ref="ModalBase" :active="active" :destroyed="destroy" :backdropDismiss="backdropDismiss">
    <div class="sm:flex sm:items-start">
      <div v-if="type!='clean'" :class="typeColors[type]" class="mx-auto flex-shrink-0 flex items-center justify-center h-12 w-12 rounded-full sm:mx-0 sm:h-10 sm:w-10">
        <IconCheck v-if="type === 'success'" class="h-5 w-5" primary="text-green-600" secondary="text-green-500" />
        <IconInfo v-if="type === 'info'" class="h-5 w-5" primary="text-blue-600" secondary="text-blue-500" />
        <IconBang v-if="type === 'danger'" class="h-5 w-5" primary="text-red-600" secondary="text-red-500" />
        <IconBang v-if="type === 'warning'" class="h-5 w-5" primary="text-yellow-600" secondary="text-yellow-500" />
      </div>
      <div class="mt-3 text-center sm:mt-0 sm:text-left" :class="{'sm:ml-4':type!=='clean'}">
        <h3 class="text-lg leading-6 font-medium text-gray-900" id="modal-headline">
          {{ title }}
        </h3>
        <div v-if="body" class="mt-2">
          <p class="text-sm leading-5 text-gray-500">
            {{ body }}
          </p>
        </div>
      </div>
    </div>
    <div v-if="fields.length" class="py-2">
      <div v-for="(field,index) in fields" v-bind:key="field.name">
        <label :for="field.name" class="block text-sm font-medium text-gray-700">{{field.label}}</label>
        <div class="mt-1">
          <input type="text" :name="field.name" v-model="field.value" :id="field.name" class="shadow-sm focus:ring-blue-500 focus:border-blue-500 block w-full sm:text-sm border-gray-300 rounded-md" :placeholder="field.placeholder?field.placeholder : ''" :required="field.required" :ref="`field${index}`" v-on:keyup.enter="index === fields.length-1 && action('primary')">
        </div>
      </div>
    </div>
    <div v-if="primary" class="mt-5 sm:mt-4 sm:flex sm:flex-row-reverse">
      <PushButton v-if="primary" ref="primary" class="flex w-full rounded-md shadow-sm sm:ml-3 sm:w-auto" :theme="primary.theme" @click.native="action('primary')">
        {{ primary.label }}
      </PushButton>
      <PushButton v-if="secondary" class="mt-3 flex w-full rounded-md shadow-sm sm:mt-0 sm:w-auto" :theme="secondary.theme" @click.native="action('secondary')">
        {{ secondary.label }}
      </PushButton>
    </div>
  </ModalBase>
</template>

<script>
  import {
    removeElement
  } from './utils.js'
  import ModalBase from './ModalBase.vue'
  import {
    PushButton
  } from 'tv-button'
  import {
    IconCheck,
    IconInfo,
    IconBang
  } from 'tv-icon'
  export default {
    components: {
      ModalBase,
      PushButton,
      IconCheck,
      IconInfo,
      IconBang
    },
    props: {
      type: {
        type: String,
        required: false,
        default: 'info',
        validate: (type) => {
          return ['success', 'info', 'danger', 'warning', 'clean'].includes(type)
        },
      },
      title: {
        type: [Boolean, String],
        required: false,
        default: false,
      },
      body: {
        type: [Boolean, String],
        required: false,
        default: false,
      },
      primary: {
        type: [Boolean, Object],
        required: false,
        default: false,
      },
      secondary: {
        type: [Boolean, Object],
        required: false,
        default: false,
      },
      fields: {
        type: [Boolean, Array],
        required: false,
        default: false,
      },
      backdropDismiss: {
        type: Boolean,
        required: false,
        default: true,
      },
      destroyed: {
        type: Function,
        required: false,
        default: () => {}
      }
    },
    data() {
      return {
        active: false,
        typeColors: {
          success: 'bg-green-100',
          info: 'bg-blue-100',
          danger: 'bg-red-100',
          warning: 'bg-yellow-100',
          clean: 'bg-transparent'
        }
      }
    },
    async mounted() {
      this.active = true
      setTimeout(() => {
        if (this.fields && this.fields.length) {
          this.$refs.field0[0].focus()
        } else if (this.primary && this.$refs.primary) {
          this.$refs.primary.$el.firstChild.focus()
        }
      }, 200)
    },
    methods: {
      async focus(index = 0) {
        document.getElementById(`modal-button-${index}`).focus()
      },
      async action(type) {
        let action = this[type].action
        let fields = {};
        this.fields && this.fields.forEach(field => {
          fields[field.name] = field.value;
        })
        if (!this[type].validate || this[type].validate(fields)) {
          this.$refs.ModalBase.destroy()
          this[type].action(fields)
        }
      },
      async destroy() {
        this.active = false
        this.$destroy()
        removeElement(this.$el)
        setTimeout(() => {
          this.destroyed()
        }, 200)
      },
    },
  }

</script>
