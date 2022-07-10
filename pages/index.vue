<template>
  <div>
    <v-card>
      <v-card-text>
        <v-row>
          <v-col cols="6">
            <v-file-input
              accept="image/*"
              label="画像"
              truncate-length="255"
              dense
              @change="onChangeImageFile"
            />
          </v-col>
          <v-col cols="6">
            <v-file-input
              accept="text/csv"
              label="pointer.csv"
              truncate-length="255"
              dense
              @change="onChangePointerCsvFile"
            />
          </v-col>
        </v-row>
      </v-card-text>
      <v-row>
        <v-col cols="xs-12 sm-12 md-7">
          <div class="image-container">
            <img ref="backgroundImage" :src="imageUrl" class="background-image">
            <div class="pointer-container" :style="pointerContainerStyle" @click.stop="onPoint">
              <VueDraggableResizable
                v-for="(p, i) in points"
                :key="i"
                :parent="true"
                :x="toPositionX(p.x)"
                :y="toPositionY(p.y)"
                :min-width="16"
                :min-height="16"
                :resizable="false"
                class="pointer"
                :style="pointerStyle(p)"
                v-html="$sanitize(p.information)"
              />
            </div>
          </div>
        </v-col>
        <v-col cols="xs-12 sm-12 md-5">
          <v-row>
            <v-col cols="auto">
              <label>color</label>
              <v-dialog
                width="max-content"
              >
                <template #activator="{ on, attrs }">
                  <v-btn
                    :color="edit.color"
                    v-bind="attrs"
                    v-on="on"
                  />
                </template>
                <v-card>
                  <v-color-picker
                    v-model="edit.color"
                    dot-size="25"
                    swatches-max-height="200"
                  />
                </v-card>
              </v-dialog>
            </v-col>
            <v-spacer />
            <v-col cols="auto">
              <VueJsonToCsv :json-data="points" csv-title="pointer">
                <v-btn :disabled="!points.length">
                  Export
                </v-btn>
              </VueJsonToCsv>
            </v-col>
          </v-row>
          <v-simple-table dense class="pointer-table">
            <template #default>
              <thead>
                <tr>
                  <th>x</th>
                  <th>y</th>
                  <th>color</th>
                  <th>information</th>
                  <th />
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(p, i) in points"
                  :key="i"
                >
                  <td>{{ percent(p.x) }}</td>
                  <td>{{ percent(p.y) }}</td>
                  <td>{{ p.color }}</td>
                  <td><input v-model="p.information" type="text"></td>
                  <td>
                    <v-btn icon @click.stop="onRemove(p)">
                      <v-icon>mdi-trash-can</v-icon>
                    </v-btn>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-col>
      </v-row>
    </v-card>
  </div>
</template>

<script>
import Vue from 'vue'
import VueDraggableResizable from 'vue-draggable-resizable'
import VueJsonToCsv from 'vue-json-to-csv'
import { parse } from 'csv-parse/lib/sync'

import sanitizeHTML from 'sanitize-html'

Vue.prototype.$sanitize = sanitizeHTML
Vue.component('VueDraggableResizable', VueDraggableResizable)

export default {
  name: 'IndexPage',

  components: {
    VueJsonToCsv
  },

  data () {
    return {
      points: [],
      imageUrl: '',
      imageWidth: 0,
      imageHeight: 0,
      edit: {
        color: 'red'
      }
    }
  },
  computed: {
    pointerContainerStyle () {
      return {
        width: `${this.imageWidth}px`,
        height: `${this.imageHeight}px`
      }
    }
  },
  methods: {
    toPositionX (x) {
      return Number.parseInt(x * this.imageWidth)
    },
    toPositionY (y) {
      return Number.parseInt(y * this.imageHeight)
    },

    percent (v) {
      return Number.parseFloat(v * 100).toFixed(2) + '%'
    },

    pointerStyle (p) {
      return {
        backgroundColor: p.color,
        width: 'max-content',
        height: 'max-content'
      }
    },
    onChangeImageFile (file) {
      const self = this
      const reader = new FileReader()
      reader.onload = () => {
        this.imageUrl = reader.result
        self.$refs.backgroundImage.onload = () => {
          self.imageWidth = self.$refs.backgroundImage.width
          self.imageHeight = self.$refs.backgroundImage.height
        }
      }
      reader.readAsDataURL(file)
    },
    onChangePointerCsvFile (file) {
      const self = this
      const reader = new FileReader()
      reader.onload = () => {
        try {
          const content = reader.result
          const records = parse(content, {
            columns: true
          })
          self.points = records
        } catch (e) {
          console.error(e)
        }
      }
      reader.readAsText(file)
    },
    onPoint (e) {
      // 画像を読み込んでいない
      if (!this.imageWidth || !this.imageHeight) {
        return
      }

      // 画像をクリックしていない
      if (e.currentTarget !== e.srcElement) {
        return
      }

      debugger

      this.points.push({
        x: e.offsetX / this.imageWidth,
        y: e.offsetY / this.imageHeight,
        color: this.edit.color,
        information: ''
      })
    },
    onRemove (p) {
      this.points = this.points.filter(point => point !== p)
    }
  }
}
</script>

<style lang="scss" scoped>
.image-container {
  position: relative;
  width: 100%;
  max-height: 90vh;
  overflow: auto;

  .pointer-container {
    position: absolute;
    top: 0;
    left: 0;
    width: max-content;
    height: max-content;
  }

  .pointer {
    cursor: pointer;
    position: absolute;
    min-width: 16px;
    min-height: 16px;
    background: red;
    border: 1px solid white;
    border-radius: 100vh;
    padding: .25rem .5rem;
    color: white;
  }
}
.pointer-table {
  th:nth-child(1), th:nth-child(2) {
    width: 2rem;
  }
  th:nth-child(3) {
    width: 3rem;
  }
  th:nth-child(5) {
    width: 2rem;
  }
  td > input {
    width: 100%;
  }
}
</style>
