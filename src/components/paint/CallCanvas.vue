<template>
  <div>
    <p>{{ brush.brushColor }}</p>
    <p>{{ brush.selectedMode }}</p>
    <v-row>
      <v-col>
        <v-color-picker
          v-model="brush.brushColor"
          class="ma-2"
        ></v-color-picker>
      </v-col>
    </v-row>
    <v-row align="center">
      <v-col cols="4" sm="3">
        <v-select
          v-model="brush.selectedMode"
          :items="items"
          item-text="name"
          item-value="kind"
          label="ペンの種類"
          return-objects
        ></v-select>
      </v-col>
      <v-col>
        <v-slider
          v-model="brush.brushSize"
          label="ブラシサイズ"
          max="100"
          min="1"
          thumb-label
        ></v-slider>
      </v-col>
    </v-row>
    <v-row class="text-center">
      <v-col>
        <v-btn-toggle v-model="toggle_controle">
          <v-btn>
            <v-icon>mdi-undo-variant</v-icon>
          </v-btn>

          <v-btn>
            <v-icon>mdi-redo-variant</v-icon>
          </v-btn>

          <v-btn @click="clearCanvas">
            <v-icon>mdi-autorenew</v-icon>
          </v-btn>

          <v-btn>
            <v-icon>mdi-content-save</v-icon>
          </v-btn>
        </v-btn-toggle>
      </v-col>
    </v-row>
    <FreeDrawing
      ref="freeDrawing"
      :mode="brush.selectedMode"
      :brushSize="brush.brushSize"
      :brushColor="brush.brushColor"
      :backgroundImage="imageFile"
    ></FreeDrawing>
  </div>
</template>

<script>
import FreeDrawing from "@/components/paint/FreeDrawing";
export default {
  name: "CallCanvas",
  components: {
    FreeDrawing
  },
  mounted() {
    this.init();
  },
  watch: {
    brush: {
      handler() {
        console.log(this.brush);
      },
      deep: true
    }
  },
  computed: {
    imageFile() {
      return require("../../assets/test.png");
    }
  },
  data() {
    return {
      items: [
        { name: "ペン", kind: "brush" },
        { name: "消しゴム", kind: "eraser" },
        { name: "直線", kind: "line" }
      ],
      toggle_do: 1,
      toggle_controle: 1,
      brush: {
        brushSize: 10,
        brushColor: "#000000FF",
        selectedMode: "brush",
        defaultMode: { name: "ペン", kind: "brush" },
        defaultBrushColor: "#000000FF"
      }
    };
  },
  methods: {
    init() {
      this.brush.selectedMode = this.brush.defaultMode.kind;
      this.brush.brushColor = this.brush.defaultBrushColor;
    },
    clearCanvas() {
      this.$refs.freeDrawing.onClearCanvas();
      this.init();
    }
  }
};
</script>
