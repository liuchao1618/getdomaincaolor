<template>
  <div
    class="rgb-base"
    :style="{background:background,width:width,height:height}"
    id="domain-image"
  >
    <img :src="imgurl" alt class="imgsr">
  </div>
</template>
<script>
  export default {
    name: 'rgb-base',
    data() {
      console.log(this.wrapSize);
      let { excludeColor, imgUrl, wrapSize } = this;
      return {
        paletteSize: '',
        exclude: excludeColor,
        imgurl: imgUrl.background,
        background: '',
        _maxLength: 10,
        width: wrapSize.width,
        height: wrapSize.height
      };
    },
    props: ['imgUrl', 'excludeColor', 'maxLength', 'wrapSize'],
    components: {},
    computed: {},
    created() {},
    methods: {
      //获取img
      getImageData(img, func) {
        let imgs = new Image();
        let imgSrc = img.src || img;
        if (imgSrc.substring(0, 5) !== 'data:') {
          imgs.crossOrigin = 'Anonymous';
        }
        imgs.onload = () => {
          //canvas
          let canvas = document.createElement('canvas');
          canvas.setAttribute('width', imgs.width);
          canvas.setAttribute('height', imgs.height);
          let context = canvas.getContext('2d');
          context.drawImage(imgs, 0, 0);
          let imageData = context.getImageData(0, 0, imgs.width, imgs.height);
          //回调
          func(imageData.data);
        };
        imgs.src = imgSrc;
      },
      createRgb(name) {
        return ['rgb(', name, ')'].join('');
      },
      frmtPobj(a, b) {
        return { name: this.createRgb(a), count: b };
      },
      //排序，b-a，按使用度最高到最低使用
      mapPalette(palette) {
        let arr = [];
        for (let prop in palette) {
          arr.push(this.frmtPobj(prop, palette[prop]));
        }
        arr.sort((a, b) => {
          return b.count - a.count;
        });
        return arr;
      },
      fitPalette(arr, fitSize) {
        if (arr.length > fitSize) {
          return arr.slice(0, fitSize);
        } else {
          for (let i = arr.length - 1; i < fitSize - 1; i++) {
            arr.push(this.frmtPobj('0,0,0', 0));
          }
          return arr;
        }
      },
      colorObj(img, opts) {
        // this.$emit('close', img);
        opts = opts || {};
        let exclude = opts.exclude || []; // 要排除的颜色:  [ '0,0,0', '255,255,255' ]
        let paletteSize = opts.maxLength || this._maxLength;
        this.getImageData(img, data => {
          let colorCounts = {};
          let rgbString = '';
          let rgb = [];
          let i = 0;
          for (; i < data.length; i += 4) {
            rgb[0] = data[i];
            rgb[1] = data[i + 1];
            rgb[2] = data[i + 2];
            rgbString = rgb.join(',');
            // 跳过未定义的数据和透明像素
            if (rgb.indexOf(undefined) !== -1 || data[i + 3] === 0) {
              continue;
            }
            // 忽略排除列表中的那些颜色。
            if (exclude.indexOf(this.createRgb(rgbString)) === -1) {
              if (rgbString in colorCounts) {
                colorCounts[rgbString] = colorCounts[rgbString] + 1;
              } else {
                colorCounts[rgbString] = 1;
              }
            }
          }
          //回调
          if (opts.success) {
            let palette = this.fitPalette(this.mapPalette(colorCounts), paletteSize + 1);
            opts.success({
              dominant: palette[0].name,
              secondary: palette[1].name,
              palette: palette
                .map(c => {
                  return c.name;
                })
                .slice(1)
            });
          }
        });
      }
    },
    mounted() {
      let { imgurl, excludeColor, maxLength } = this;
      this.colorObj(imgurl, {
        exclude: excludeColor,
        maxLength,
        success: payload => {
          this.background = payload.dominant;
        }
      });
    }
  };
</script>
<style scoped>
  body,
  html {
    height: 100%;
  }
  .rgb-base {
    position: relative;
    /* height: 100%;
    width: 100%; */
  }
  #domain-image .imgsr {
    width: 100%;
    height: 100%;
    /* position: absolute;
    max-height: 100% !important;
    max-width: 100%;
    width: 100% !important;
    height: 100% !important;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto; */
  }
</style>
