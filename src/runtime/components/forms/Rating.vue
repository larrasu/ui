<template>
  <div :class="wrapperClass">
    <UIcon
      v-for="star of [...Array(max).keys()].slice().reverse()"
      :key="star"
      :name="icon"
      :class="[
        ratingClass,
        (rate >= star + 1 ? ui.active : ui.inactive).replaceAll('{color}', color),
        'order-{order}'.replaceAll('{order}', star.toString()),
      ]"
      @click="setRating(star + 1)"
    />
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, toRef } from 'vue'
import type { PropType } from 'vue'
import UIcon from '../elements/Icon.vue'
import { twJoin, twMerge } from 'tailwind-merge'
import { useFormGroup } from '../../composables/useFormGroup'
import { mergeConfig } from '../../utils'
import { useUI } from '../../composables/useUI'
import type { Strategy } from '../../types'
// @ts-expect-error
import appConfig from '#build/app.config'
import { rating } from '#ui/ui.config'
import colors from '#ui-colors'

const config = mergeConfig<typeof rating>(appConfig.ui.strategy, appConfig.ui.rating, rating)

export default defineComponent({
  components: {
    UIcon
  },
  props: {
    modelValue: {
      type: Number,
      default: 0
    },
    icon: {
      type: String,
      default: () => config.default.icon
    },
    max: {
      type: Number,
      default: 5,
      validate (value: number) {
        return value >= 1 && value < Number.MAX_VALUE
      }
    },
    color: {
      type: String as PropType<typeof colors[number]>,
      default: () => config.default.color,
      validator (value: string) {
        return appConfig.ui.colors.includes(value)
      }
    },
    size: {
      type: String as PropType<keyof typeof config.size>,
      default: () => config.default.size
    },
    readOnly: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    class: {
      type: [String, Object, Array] as PropType<any>,
      default: undefined
    },
    ui: {
      type: Object as PropType<Partial<typeof config & { strategy?: Strategy }>>,
      default: undefined
    }
  },
  emits: ['update:modelValue'],
  setup (props, { emit }) {
    const { ui } = useUI('rating', toRef(props, 'ui'), config, toRef(props, 'class'))

    const { emitFormBlur, size, color } = useFormGroup(props)

    const rate = computed({
      get () {
        return props.modelValue
      },
      set (value) {
        emit('update:modelValue', value)
        emitFormBlur()
      }
    })

    const setRating = (rating: number) => {
      if (props.readOnly || props.disabled) return

      if (rate.value == rating) {
        rate.value = 0
        return
      }

      rate.value = rating
    }

    const wrapperClass = computed(() => {
      return twMerge(twJoin(
        ui.value.wrapper,
        ui.value.size[size.value]
      ), props.class)
    })

    const ratingClass = computed(() => {
      return twJoin(
        ui.value.base,
        ui.value.size[props.size],
        props.readOnly ? ui.value.readonly : '',
        props.disabled ? ui.value.disabled : '',
        !props.readOnly && !props.disabled ? ui.value.hover.replaceAll('{color}', color.value) : ''
      )
    })

    return {
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      rate,
      setRating,
      wrapperClass,
      ratingClass
    }
  }
})
</script>
