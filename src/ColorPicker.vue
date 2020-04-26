<template>
  <div class="colorpicker" ref="colorpicker" v-bind:class="{opened: opened}">
    <div class="current-color" @click="toggleOpen()">
      <div class="current-color-inner" v-bind:style="{'background-color':color}"></div>
    </div>
    <div class="colorpicker-right">
      <div class="hues">
        <table class="colors-container noselect" @mousedown="updatableHue = true" @mouseleave="updatableHue = false" @mouseup="updatableHue = false">
          <tr>
            <td class="color-unit" v-for="(color, idx) in hues" @click="updateHue(idx, true)" @mouseover="updateHue(idx)"  v-bind:key="idx" v-bind:style="{'background-color': color}">
              <div class="color-unit-current" v-if="idx == currentHue"></div>
            </td>
          </tr>
        </table>
        <table class="colors-container saturation noselect" @mousedown="updatableSaturation = true" @mouseleave="updatableSaturation = false" @mouseup="updatableSaturation = false">
          <tr>
            <td class="color-unit" v-for="(color, idx) in saturations" @click="updateSaturation(idx, true)" @mouseover="updateSaturation(idx)"  v-bind:key="idx" v-bind:style="{'background-color': color}">
              <div class="color-unit-current" v-if="idx == currentSaturation"></div>
            </td>
          </tr>
        </table>
        <table class="colors-container lightness noselect" @mousedown="updatableLightness = true" @mouseleave="updatableLightness = false" @mouseup="updatableLightness = false">
          <tr>
            <td class="color-unit" v-for="(color, idx) in lightness" @click="updateLightness(idx, true)" @mouseover="updateLightness(idx)"  v-bind:key="idx" v-bind:style="{'background-color': color}">
              <div class="color-unit-current" v-if="idx == currentLightness"></div>
            </td>
          </tr>
        </table>
        <table class="colors-container alpha noselect" @mousedown="updatableAlpha = true" @mouseleave="updatableAlpha = false" @mouseup="updatableAlpha = false">
          <tr>
            <td class="color-unit" v-for="(alpha, idx) in alphas" @click="updateAlpha(idx, true)" @mouseover="updateAlpha(alpha)"  v-bind:key="idx" v-bind:style="{'background-color': 'hsla('+currentHue+','+currentSaturation+'%,'+currentLightness+'%, '+alpha+')'}">
              <div class="color-unit-current" v-if="alpha == currentAlpha"></div>
            </td>
          </tr>
        </table>
      </div>
      <input class="color-input" v-model="color" @input="updateColorBasedOnInput()"/>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'ColorPicker',
  props: {
    value : String
  },
  watch : {
    color(value) {
      this.$emit('input', value);
        // or generate/simulate a native events (not sure how, but its outside Vue's realm I think
    },
  },
  data: () => {
    return {
      opened: false,
      color: '',
      updatableHue: false,
      currentHue: 0,
      updatableSaturation: false,
      currentSaturation: 0,
      updatableLightness: false,
      currentLightness: 0,
      updatableAlpha: false,
      currentAlpha: 0,
      hues: [],
      alphas: [],
      saturations: [],
      lightness: [],
    }
  },
  mounted: function (){
    this.bindClickOutside();
    this.color = this.value;
    if (this.color == undefined) {
      this.color = "#000000"
    }
    for (var i = 0; i <= 360; i++) {
      this.hues.push('hsl('+i+',100%,50%)');
    }
    for (var a = 0; a <= 100; a++) {
      this.alphas.push(a / 100);
    }
    this.updateColorBasedOnInput();
  },
  methods: {
    bindClickOutside: function() {
      var that = this;
      document.addEventListener('click', function(event) {
        var isClickInside = that.$refs.colorpicker.contains(event.target);
        if (!isClickInside && that.opened) {
          that.opened = false;
        }
      });
    },
    updateColorBasedOnInput: function () {
      let parsedColor = this.parseColor(this.color);
      if (parsedColor == undefined) {
        parsedColor = [0,0,0,1];
      }
      let hslColor = this.rgbToHsl(parsedColor[0], parsedColor[1], parsedColor[2]);
      this.currentHue = parseInt(hslColor[0] * 360);
      this.currentSaturation = parseInt(hslColor[1] * 100);
      this.currentLightness = parseInt(hslColor[2] * 100);
      this.currentAlpha = parseInt(parsedColor[3] * 100) / 100;
      this.updateSaturationsAndLightness();
    },
    toggleOpen:function () {
      this.opened = !this.opened;
    },
    parseColor:function (color) {
      function _hslToRgb(hsl){
        if(typeof hsl== 'string'){
            hsl= hsl.match(/(\d+(\.\d+)?)/g);
        }
        var h= hsl[0]/360, s= hsl[1]/100, l= hsl[2]/100, a = hsl[3]===undefined?1:hsl[3], t1, t2, t3, rgb, val;
        if(s== 0){
            val= Math.round(l*255);
            rgb= [val, val, val, a];
        }
        else{
            if(l<0.5)
                t2= l*(1 + s);
            else
                t2= l + s - l*s;
            t1 = 2*l - t2;
            rgb = [0, 0, 0];
            for(var i=0; i<3; i++){
                t3 = h + 1/3 * -(i - 1);
                t3 < 0 && t3++;
                t3 > 1 && t3--;
                if (6 * t3 < 1)
                    val= t1 + (t2 - t1) * 6 * t3;
                else if (2 * t3 < 1)
                    val= t2;
                else if (3*t3<2)
                    val = t1 + (t2 - t1) * (2 / 3 - t3) * 6;
                else
                    val= t1;
                rgb[i] = Math.round(val*255);
            }
        }
        rgb.push(a);
        return rgb;
    }
    var _colorsByName = {aliceblue:"#f0f8ff",antiquewhite:"#faebd7",aqua:"#00ffff",aquamarine:"#7fffd4",azure:"#f0ffff",beige:"#f5f5dc",bisque:"#ffe4c4",
        black:"#000000",blanchedalmond:"#ffebcd",blue:"#0000ff",blueviolet:"#8a2be2",brown:"#a52a2a",burlywood:"#deb887",cadetblue:"#5f9ea0",
        chartreuse:"#7fff00",chocolate:"#d2691e",coral:"#ff7f50",cornflowerblue:"#6495ed",cornsilk:"#fff8dc",crimson:"#dc143c",cyan:"#00ffff",
        darkblue:"#00008b",darkcyan:"#008b8b",darkgoldenrod:"#b8860b",darkgray:"#a9a9a9",darkgreen:"#006400",darkkhaki:"#bdb76b",
        darkmagenta:"#8b008b",darkolivegreen:"#556b2f",darkorange:"#ff8c00",darkorchid:"#9932cc",darkred:"#8b0000",darksalmon:"#e9967a",
        darkseagreen:"#8fbc8f",darkslateblue:"#483d8b",darkslategray:"#2f4f4f",darkturquoise:"#00ced1",darkviolet:"#9400d3",deeppink:"#ff1493",
        deepskyblue:"#00bfff",dimgray:"#696969",dodgerblue:"#1e90ff",firebrick:"#b22222",floralwhite:"#fffaf0",forestgreen:"#228b22",
        fuchsia:"#ff00ff",gainsboro:"#dcdcdc",ghostwhite:"#f8f8ff",gold:"#ffd700",goldenrod:"#daa520",gray:"#808080",green:"#008000",
        greenyellow:"#adff2f",honeydew:"#f0fff0",hotpink:"#ff69b4",indianred :"#cd5c5c",indigo :"#4b0082",ivory:"#fffff0",khaki:"#f0e68c",
        lavender:"#e6e6fa",lavenderblush:"#fff0f5",lawngreen:"#7cfc00",lemonchiffon:"#fffacd",lightblue:"#add8e6",lightcoral:"#f08080",
        lightcyan:"#e0ffff",lightgoldenrodyellow:"#fafad2",lightgray:"#d3d3d3",lightgreen:"#90ee90",lightpink:"#ffb6c1",lightsalmon:"#ffa07a",
        lightseagreen:"#20b2aa",lightskyblue:"#87cefa",lightslategray:"#778899",lightsteelblue:"#b0c4de",lightyellow:"#ffffe0",lime:"#00ff00",
        limegreen:"#32cd32",linen:"#faf0e6",magenta:"#ff00ff",maroon:"#800000",mediumaquamarine:"#66cdaa",mediumblue:"#0000cd",
        mediumorchid:"#ba55d3",mediumpurple:"#9370db",mediumseagreen:"#3cb371",mediumslateblue:"#7b68ee",mediumspringgreen:"#00fa9a",
        mediumturquoise:"#48d1cc",mediumvioletred:"#c71585",midnightblue:"#191970",mintcream:"#f5fffa",mistyrose:"#ffe4e1",moccasin:"#ffe4b5",
        navajowhite:"#ffdead",navy:"#000080",oldlace:"#fdf5e6",olive:"#808000",olivedrab:"#6b8e23",orange:"#ffa500",orangered:"#ff4500",
        orchid:"#da70d6",palegoldenrod:"#eee8aa",palegreen:"#98fb98",paleturquoise:"#afeeee",palevioletred:"#db7093",papayawhip:"#ffefd5",
        peachpuff:"#ffdab9",peru:"#cd853f",pink:"#ffc0cb",plum:"#dda0dd",powderblue:"#b0e0e6",purple:"#800080",red:"#ff0000",rosybrown:"#bc8f8f",
        royalblue:"#4169e1",saddlebrown:"#8b4513",salmon:"#fa8072",sandybrown:"#f4a460",seagreen:"#2e8b57",seashell:"#fff5ee",sienna:"#a0522d",
        silver:"#c0c0c0",skyblue:"#87ceeb",slateblue:"#6a5acd",slategray:"#708090",snow:"#fffafa",springgreen:"#00ff7f",steelblue:"#4682b4",
        tan:"#d2b48c",teal:"#008080",thistle:"#d8bfd8",tomato:"#ff6347",turquoise:"#40e0d0",violet:"#ee82ee",wheat:"#f5deb3",white:"#ffffff",
        whitesmoke:"#f5f5f5",yellow:"#ffff00",yellowgreen:"#9acd32"
    };
        
        color = color.trim().toLowerCase();
        color = _colorsByName[color] || color;
        var hex3 = color.match(/^#([0-9a-f]{3})$/i);
        if (hex3) {
            hex3 = hex3[1];
            return [
                parseInt(hex3.charAt(0),16)*0x11,
                parseInt(hex3.charAt(1),16)*0x11,
                parseInt(hex3.charAt(2),16)*0x11, 1
            ];
        }
        var hex6 = color.match(/^#([0-9a-f]{6})$/i);
        if (hex6) {
            hex6 = hex6[1];
            return [
                parseInt(hex6.substr(0,2),16),
                parseInt(hex6.substr(2,2),16),
                parseInt(hex6.substr(4,2),16), 1
            ];
        }
        var rgba = color.match(/^rgba\s*\(\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+.*\d*)\s*\)$/i) || color.match(/^rgba\s*\(\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+)\s*\)$/i);
        if( rgba ) {
            return [rgba[1],rgba[2],rgba[3], rgba[4]===undefined?1:rgba[4]];
        }
        var rgb = color.match(/^rgb\s*\(\s*(\d+)\s*,\s*(\d+)\s*,\s*(\d+)\s*\)$/i);
        if( rgb ) {
            return [rgb[1],rgb[2],rgb[3],1];
        }
        if(color.indexOf('hsl')== 0)
            return _hslToRgb(color);
    },
    hslToRgb: function(h, s, l){
      var r, g, b;

      if(s == 0){
          r = g = b = l; // achromatic
      }else{
          var hue2rgb = function hue2rgb(p, q, t){
              if(t < 0) t += 1;
              if(t > 1) t -= 1;
              if(t < 1/6) return p + (q - p) * 6 * t;
              if(t < 1/2) return q;
              if(t < 2/3) return p + (q - p) * (2/3 - t) * 6;
              return p;
          }

          var q = l < 0.5 ? l * (1 + s) : l + s - l * s;
          var p = 2 * l - q;
          r = hue2rgb(p, q, h + 1/3);
          g = hue2rgb(p, q, h);
          b = hue2rgb(p, q, h - 1/3);
      }

      return [Math.round(r * 255), Math.round(g * 255), Math.round(b * 255)];
    },
    rgbToHsl: function(r, g, b){
        r /= 255, g /= 255, b /= 255;
        var max = Math.max(r, g, b), min = Math.min(r, g, b);
        var h, s, l = (max + min) / 2;

        if(max == min){
            h = s = 0; // achromatic
        }else{
            var d = max - min;
            s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
            switch(max){
                case r: h = (g - b) / d + (g < b ? 6 : 0); break;
                case g: h = (b - r) / d + 2; break;
                case b: h = (r - g) / d + 4; break;
            }
            h /= 6;
        }

        return [h, s, l];
    },
    updateSaturationsAndLightness: function () {
      this.saturations = [];
      for (var s = 0; s <= 100; s++) {
        this.saturations.push('hsl('+this.currentHue+','+s+'%,50%)');
      }
      this.lightness = [];
      for (var l = 0; l <= 100; l++) {
        this.lightness.push('hsl('+this.currentHue+',100%,'+l+'%)');
      }
    },
    updateHue: _.throttle(function(hue, forced) {
      if (!this.updatableHue && forced !== true) {
        return ;
      }
      this.currentHue = hue;
      this.updateSaturationsAndLightness();
      this.updateColor();
    }, 10),
    updateSaturation: _.throttle(function(saturation, forced) {
      if (!this.updatableSaturation && forced !== true) {
        return ;
      }
      this.currentSaturation = saturation;
      this.updateColor();
    }, 10),
    updateLightness: _.throttle(function(lightness, forced) {
      if (!this.updatableLightness && forced !== true) {
        return ;
      }
      this.currentLightness = lightness;
      this.updateColor();
    }, 10),
    updateAlpha: _.throttle(function(alpha, forced) {
      if (!this.updatableAlpha && forced !== true) {
        return ;
      }
      this.currentAlpha = alpha;
      this.updateColor();
    }, 10),
    updateColor: function() {
      var rgbColor = this.hslToRgb(this.currentHue / 360, this.currentSaturation / 100, this.currentLightness / 100);
      var strColor = 'rgba(' + rgbColor[0] + ',' + rgbColor[1] + ',' + rgbColor[2] + ', '+this.currentAlpha+')'
      this.color = strColor;
    },  
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  * {
    box-sizing: border-box;
  }

  input {
    background: #fff;
    border: 1px solid #e1e1e1;
    padding: 8px 16px;
    border-radius: 8px;
    width: 100%;
    outline: none;
  }

  table {
    border-collapse: collapse;
  }
  .colorpicker {
    position:relative;
    text-align: left;
  }

  .colorpicker-right {
    position: absolute;
    top: 0px;
    left: 62px;
    background: white;
    padding: 24px;
    -webkit-box-shadow: 2px 3px 8px 0px #58585852;
    box-shadow: 2px 3px 8px 0px #58585852;
    margin-bottom: 24px;
    z-index: 1;
  }

  .colorpicker:not(.opened) .colorpicker-right {
    display: none;
  }
  
  .color-input {
    margin-top:1rem;
    width: 320px!important;
    max-width: 100%;
  }

  .current-color {
    margin-right:1rem;
    border-radius:42px;
    border:solid 1px #e1e1e1;
    background-image: url('alpha.png');
    background-position: 0 100%;
    display:inline-block;
  }

  .current-color:hover {
    cursor:pointer;
  }

  .current-color-inner {
    width:42px;
    min-height:42px;
    border-radius:42px;
  }

  .saturation, .lightness, .alpha {
    margin-top:1rem;
  }

  .alpha {
    background-image: url('alpha.png');
    background-position: 0 100%;
  }

  .colors-container {
    height: 16px;
    width: 320px;
    max-width: 100%;
  }
  
  .colors-container:hover {
    cursor:pointer;
  }

  .color-unit {
    padding: 0;
    position:relative;
  }

  .color-unit-current {
    position:absolute;
    top:100%;
    background:black;
    height:3px;
    width:8px;
    left:-4px;
  }

  .noselect {
    -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
    -khtml-user-select: none; /* Konqueror HTML */
    -moz-user-select: none; /* Old versions of Firefox */
    -ms-user-select: none; /* Internet Explorer/Edge */
    user-select: none; /* Non-prefixed version, currently
                       supported by Chrome, Opera and Firefox */
  }

</style>
