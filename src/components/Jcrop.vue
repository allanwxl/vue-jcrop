<template>
  <div v-once></div>
</template>

<script>
import Jcrop from '../jcrop';

export default {
  name: 'Jcrop',
  props: [ 'src', 'options', 'rect', 'initRects' ],
  mounted: function() {
    const img = new Image();
    this.$el.appendChild(img);
    img.src = this.src;
    Jcrop.load(img).then(this.startup);
  },
  methods: {
    startup(img){
      console.log(this.options);
      this.jcrop = Jcrop.attach(img,this.options||{});
      let rect = Jcrop.Rect.sizeOf(this.jcrop.el);

      ['activate','update','change','remove'].forEach(ev => {
        this.jcrop.listen('crop.'+ev,widget => this.$emit(ev,widget));
      });

      if (this.rect) {
        rect = Jcrop.Rect.from(this.rect);
        this.jcrop.newWidget(rect);
      }
      // else rect = rect.scale(.7,.5).center(rect.w,rect.h)
      if (this.initRects && this.initRects.length > 0) {
        this.initRects.forEach( el => {
          rect = Jcrop.Rect.from(el);
          this.jcrop.newWidget(rect);
        })
      }
      this.pos = this.jcrop.pos;
      this.jcrop.focus();
    }
  },
  watch: {
    initRects: {
      handler: function(v) {
        this.jcrop.destroy();
        const img = new Image();
        this.$el.appendChild(img);
        img.src = this.src;
        Jcrop.load(img).then(this.startup);
      },
      deep: true
    },
    rect: {
      handler: function(v) {
        if (!this.jcrop.active) return false;
        this.jcrop.active.animate(Jcrop.Rect.from(v),20,'inOutCirc')
          .then(() => {
            this.jcrop.focus();
          });
      },
      deep: true
    },
    options: {
      handler: function(o) { this.jcrop.setOptions(o); },
      deep: true
    }
  },
  beforeDestroy: function () {
    this.jcrop.destroy();
  },
  data: () => ({
    pos: null,
    jcrop: null
  })
}
</script>

<style lang="scss">
@import "../Jcrop/build/css/jcrop.scss";
</style>
