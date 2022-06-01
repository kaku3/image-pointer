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
        <v-col cols="7">
          <div class="image-container">
            <img ref="backgroundImage" :src="imageUrl" class="background-image">
            <div class="pointer-container" :style="pointerContainerStyle" @mousedown.stop="onPoint">
              <VueDraggableResizable
                v-for="(p, i) in points"
                :key="i"
                :parent="true"
                :x="p.x"
                :y="p.y"
                :w="16"
                :h="16"
                :resizable="false"
                class="pointer"
                :style="pointerStyle(p)"
              />
            </div>
          </div>
        </v-col>
        <v-col cols="5">
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
            <v-col cols="4">
              <v-slider
                v-model="edit.size"
                dense
                label="size"
                max="64"
                min="4"
              >
                <template #append>
                  <v-text-field
                    v-model="edit.size"
                    class="mt-0 pt-0"
                    hide-details
                    single-line
                    type="number"
                    style="width: 60px"
                  />
                </template>
              </v-slider>
            </v-col>
            <v-spacer />
            <v-col cols="2">
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
                  <th>size</th>
                  <th>information</th>
                  <th />
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(p, i) in points"
                  :key="i"
                >
                  <td>{{ p.x }}</td>
                  <td>{{ p.y }}</td>
                  <td>{{ p.size }}</td>
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
        color: 'red',
        size: 16
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
    pointerStyle (p) {
      return {
        backgroundColor: p.color,
        width: `${p.size}px`,
        height: `${p.size}px`
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
      this.points.push({
        x: e.offsetX,
        y: e.offsetY,
        size: this.edit.size,
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
    background: red;
    border: 1px solid white;
    border-radius: 100vh;
    opacity: .5;
  }
}
.pointer-table {
  th:nth-child(1), th:nth-child(2) {
    width: 2rem;
  }
  th:nth-child(3) {
    width: 3rem;
  }
  th:nth-child(4) {
    width: 2rem;
  }
  th:nth-child(6) {
    width: 2rem;
  }
  td > input {
    width: 100%;
  }
}
</style>
