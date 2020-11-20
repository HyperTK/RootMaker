<template>
  <div>
    <div ref="container">
      <canvas :width="width / 2" :height="height / 2" ref="canvas"></canvas>
    </div>
  </div>
</template>
<script>
import Konva from "konva";

export default {
  name: "FreeDrawing",
  props: {
    mode: {
      type: String,
      default: "",
    },
    brushColor: {
      type: String,
      default: "",
    },
    backgroundImage: {
      type: String,
      default: "",
    },
    brushSize: {
      type: Number,
      default: 10
    }
  },
  data() {
    return {
      width: window.innerWidth,
      height: window.innerHeight,
      stage: null,
      canvas: null,
      context: null,
      drawingLayer: null,
      drawingScope: null,
      imageObj: null,
      backgroundLayer: null,
      backgroundImageScope: null,
      lastPointerPosition: {},
      localPos: {
        x: 0,
        y: 0,
      },
      pos: null,
      isPaint: false,
    };
  },
  watch: {
    brushColor() {
      this.context.strokeStyle = this.brushColor;
    },
    brushSize() {
      this.context.lineWidth = this.brushSize;
    },
    mode() {
      console.log(this.mode);
    },
  },
  mounted() {
    var container = this.$refs.container;
    // ステージを用意
    this.stage = new Konva.Stage({
      container,
      width: this.width,
      height: this.height,
    });
    // レイヤーを用意
    this.drawingLayer = new Konva.Layer();
    // ステージにレイヤーを追加
    this.stage.add(this.drawingLayer);

    this.canvas = this.$refs.canvas;
    this.drawingScope = new Konva.Image({
      image: this.canvas,
      x: this.width / 4,
      y: 5,
      stroke: "black",
    });
    this.drawingLayer.add(this.drawingScope);
    // ステージを描画する
    this.stage.draw();

    // 画像読み込み
    this.imageObj = new Image();

    this.context = this.canvas.getContext("2d");
    this.context.strokeStyle = this.brushColor;
    this.context.lineJoin = "round";
    this.context.lineWidth = this.brushSize;
    // イベント追加
    this.drawingScope.on("mousedown", this.mousedown);
    this.stage.addEventListener("mouseup", this.mouseup);
    this.stage.addEventListener("mousemove", this.mousemove);
    this.drawingScope.on("touchstart", this.mousedown);
    this.stage.addEventListener("touchend", this.mouseup);
    this.stage.addEventListener("touchmove", this.mousemove);
    this.imageObj.addEventListener("load", this.imageOnLoad);

    this.imageObj.src = this.backgroundImage;
  },
  methods: {
    mousedown() {
      this.isPaint = true;
      // マウスダウン時の座標を取得
      this.lastPointerPosition = this.stage.getPointerPosition();
    },
    mouseup() {
      this.isPaint = false;
    },
    mousemove() {
      if (!this.isPaint) {
        return;
      }
      // ペンモード
      if (this.isTargetMode("brush") || this.isTargetMode("line")) {
        this.context.globalCompositeOperation = "source-over";
      }
      // 消しゴムモード
      if (this.isTargetMode("eraser")) {
        this.context.globalCompositeOperation = "destination-out";
      }

      this.context.beginPath();

      this.localPos.x = this.lastPointerPosition.x - this.drawingScope.x();
      this.localPos.y = this.lastPointerPosition.y - this.drawingScope.y();

      // 描画開始座標を指定する
      this.context.moveTo(this.localPos.x, this.localPos.y);

      this.pos = this.stage.getPointerPosition();
      this.localPos.x = this.pos.x - this.drawingScope.x();
      this.localPos.y = this.pos.y - this.drawingScope.y();

      // 描画開始座標からlineToに指定された座標まで描画する
      this.context.lineTo(this.localPos.x, this.localPos.y);
      this.context.closePath();
      this.context.stroke();
      this.drawingLayer.draw();

      this.lastPointerPosition = this.pos;
    },
    imageOnLoad() {
      this.backgroundLayer = new Konva.Layer();

      // 背景イメージ(xとy座標はthis.drawingScopeと同じにする)
      this.backgroundImageScope = new Konva.Image({
        image: this.imageObj,
        x: this.width / 4,
        y: 5,
        width: this.canvas.width,
        height: this.canvas.height,
      });
      // this.backgroundImageScope.cache();
      // this.backgroundImageScope.filters([Konva.Filters.Grayscale]);

      // 背景レイヤーにイメージを追加
      this.backgroundLayer.add(this.backgroundImageScope);
      this.stage.add(this.backgroundLayer);

      // 背景イメージを最背面に移動
      this.backgroundLayer.moveToBottom();
    },
    onClearCanvas() {
      this.context.globalCompositeOperation = "destination-out";
      this.context.fillRect(0, 0, this.width, this.height);
      this.drawingLayer.draw();

      this.$emit("on-reset");
    },
    isTargetMode(targetMode) {
      console.log(targetMode, this.mode)
      // 現在のモードが指定されたモードと一致するか
      return this.mode === targetMode;
    },
  },
};
</script>
