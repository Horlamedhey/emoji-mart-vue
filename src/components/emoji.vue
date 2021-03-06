<template>

<span class="emoji-mart-emoji" @mouseenter="onMouseEnter" @mouseleave="onMouseLeave" @click="onClick">
  <span v-if="isCustom" :title="title" :style="customEmojiStyles"></span>
  <span v-else-if="isNative" :title="title" :style="nativeEmojiStyles">{{ nativeEmoji }}</span>
  <span v-else-if="hasEmoji" :title="title" :style="fallbackEmojiStyles"></span>
  <span v-else>{{ fallbackEmoji }}</span>
</span>

</template>

<script>

import data from '../data'
import { getData, getSanitizedData, unifiedToNative } from '../utils'

const SHEET_COLUMNS = 52

export default {
  props: {
    backgroundImageFn: {
      type: Function,
      default: function(set, sheetSize) {
        return `https://unpkg.com/emoji-datasource-${set}@${EMOJI_DATASOURCE_VERSION}/img/${set}/sheets/${sheetSize}.png`
      }
    },
    native: {
      type: Boolean,
      default: false
    },
    forceSize: {
      type: Boolean,
      default: false
    },
    tooltip: {
      type: Boolean,
      default: false
    },
    fallback: {
      type: Function
    },
    skin: {
      type: Number,
      default: 1
    },
    sheetSize: {
      type: Number,
      default: 64
    },
    set: {
      type: String,
      default: 'apple'
    },
    size: {
      type: Number,
      default: 24
    },
    emoji: {
      type: [String, Object],
      required: true
    }
  },
  computed: {
    data() {
      return getData(this.emoji, this.skin, this.set)
    },
    sanitizedData() {
      return getSanitizedData(this.emoji, this.skin, this.set)
    },
    isNative() {
      return this.native
    },
    isCustom() {
      return this.data.custom
    },
    hasEmoji() {
      return this.data['has_img_' + this.set]
    },
    nativeEmoji() {
      if (this.data.unified) {
        return unifiedToNative(this.data.unified)
      } else {
        return ''
      }
    },
    fallbackEmoji() {
      return this.fallback ? this.fallback(this.emoji) : null
    },
    nativeEmojiStyles() {
      let styles = { fontSize: this.size + 'px' }

      if (this.forceSize) {
        styles.display = 'inline-block'
        styles.width = this.size + 'px'
        styles.height = this.size + 'px'
      }

      return styles
    },
    fallbackEmojiStyles() {
      if (this.isCustom) {
        return this.customEmojiStyles
      } else if (this.hasEmoji) {
        return {
          display: 'inline-block',
          width: this.size + 'px',
          height: this.size + 'px',
          backgroundImage: 'url(' + this.backgroundImageFn(this.set, this.sheetSize) + ')',
          backgroundSize: (100 * SHEET_COLUMNS) + '%',
          backgroundPosition: this.getPosition()
        }
      } else {
        return null
      }
    },
    customEmojiStyles() {
      return {
        display: 'inline-block',
        width: this.size + 'px',
        height: this.size + 'px',
        backgroundImage: 'url(' + this.data.imageUrl + ')',
        backgroundSize: '100%',
      }
    },
    title() {
      return this.tooltip ? this.data.short_names[0] : null
    }
  },
  methods: {
    getPosition() {
      let multiply = 100 / (SHEET_COLUMNS - 1),
          x = multiply * this.data.sheet_x,
          y = multiply * this.data.sheet_y

      return `${x}% ${y}%`
    },
    onClick() {
      this.$emit('click', this.sanitizedData)
    },
    onMouseEnter() {
      this.$emit('mouseenter', this.sanitizedData)
    },
    onMouseLeave() {
      this.$emit('mouseleave', this.sanitizedData)
    }
  }
}

</script>

<style scoped>

.emoji-mart-emoji {
  position: relative;
  display: inline-block;
  font-size: 0;
}

</style>
