<template>
  <td class="divider"
    :class="{isHover: hoverDevIndex === index}"
    @mouseover="hoverDev(index)"
    @mouseout="hoverDev()"
    @mousedown="event => moveDevStart(event, index)"></td>
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
  name: 'divider',
  props: {
    'index': { type: Number, required: true },
    'prop': { type: String, required: true }
  },
  methods: {
    ...mapActions([
      'setProperty'
    ]),
    hoverDev (index) {
      if (this.movingIndex === -1) {
        this.setProperty({
          property: `private.display.${this.prop}.hoverDevIndex`,
          value: index !== undefined ? index : -1,
          logOff: true
        })
      }
    },
    moveDevStart (event, index) {
      this.setProperty({
        property: `private.display.${this.prop}`,
        value: {
          movingIndex: index,
          startX: event.clientX,
          startLeftWidth: this.widthList[index],
          startRightWidth: this.widthList[index + 1]
        },
        logOff: true
      })
    }
  },
  computed: mapState({
    widthList (state) {
      return state.private.display[this.prop].widthList
    },
    hoverDevIndex (state) {
      return state.private.display[this.prop].hoverDevIndex
    },
    movingIndex (state) {
      return state.private.display[this.prop].movingIndex
    }
  })
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.divider {
  background-color: rgb(183, 186, 188);
  cursor: col-resize;
  position: relative;
  width: 1px;
}
.divider:after {
  position: absolute;
  height: 100%;
  top: 0;
  left: -3px;
  content: '';
  width: 7px;
  /* background-color: red; */
}
</style>
