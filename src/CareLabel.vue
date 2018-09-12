<template lang='pug'>
div
  div#viewarea
    label 幅(cm)
    input(type="number" v-model="width" placeholder="幅" min=0 max=99)
    label 高さ(cm)
    input(type="number" v-model="height" placeholder="高さ" min=0 max=99)

    div#view(:style="viewsize")
      div(v-for="elem in parsed")

        div(v-if="check(elem, /^@mixings/)" :class="classname(elem)")
          table
            tr(v-for="mixing in take(elem)")
              td(v-for="i in columns(take(elem))") {{ mixing[i-1] || "" }}
  
        div(v-else-if="check(elem, /^@marks/)" :class="classname(elem)")
          div(v-for="mark in take(elem)")
            img(v-if="isValidMarkId(mark[0])" :src="`/img/${mark[0]}.jpg`")
  
        div(v-else-if="check(elem, /^@/)" :class="classname(elem)")
          div(v-for="e in take(elem)") {{ e.join(' ') }}

  div#editor
    textarea(v-model="text")
    textarea(:value="preserved" readonly)
    textarea(:value="JSON.stringify(parsed)" readonly)
    textarea(v-model="style" @input="applyStyle")
        

</template>
<script>
import parser from './parser.js'

const initialText =
`@number
  18AKN01575
@title
  サイズ
@size
  M
@title
  品質表示
@mixings
  表地　 綿  100%
  裏地　 綿  50%
  　 ナイロン  50%
  　
@title
  取り扱い方
@marks
  100 やさしく
  200
  300
  410
  500
  600
  700
@notes
  もみ洗いをしないでください。
  白色及び淡色物と一緒に洗わないでください。
@address
  シタテル株式会社
  096-285-7519
`
const initialStyle =
`#view {
  font-size: 4mm;
}

#view > div {
  margin: 1mm;
}

.number {
  text-align: center;
}

.title {
  text-align: center;
}

.size {
  text-align: center;
  font-size: 8mm;
  font-weight: bold;
}

.mixings table {
  width: 100%
}

.mixings table tr td {
  font-size: 4mm;
  line-height: 4mm;
}

.mixings table tr td:last-child {
  text-align: right;
}

.marks {
  text-align: center;
}

.marks div {
  display: inline;
}

.marks div img {
  width: 11mm;
  height: 11mm;
  object-fit: contain;
}

.notes {
  margin-top: 4mm;
}

.notes div::before {
  content: '・';
}
.notes div {
  font-size: 3mm;
  padding-left: 1em;
  text-indent: -1em;
}

.address {
  margin-top: 8mm;
  text-align: center;
  font-size: 4mm;
  line-height: 4mm;
}
`

const validMarks = ['100','130','132','141','150','160','170','200','220','310','410','420','430','440','500','520','600','611','621','710','712','110','131','140','142','151','161','190','210','300','320','415','425','435','445','510','530','610','620','700','711']

export default {
  data() {
    return {
      text: initialText,
      validText: initialText,
      style: initialStyle,
      width: 5,
      height: 20
    }
  },
  computed: {
    preserved() {
      return parser.preserve(this.text)
    },
    parsed() {
      const obj = parser.parse(this.text)
      if (obj) this.validText = this.text
      return obj || parser.parse(this.validText)
    },
    viewsize() {
      return {
        'width'  : this.width + 'cm',
        'height' : this.height + 'cm'
      }
    }
  },
  methods: {
    check(elem, id) {
      return elem && Array.isArray(elem) && 1 < elem.length && elem[0].match(id)
    },
    take(elem) {
      return elem[1]
    },
    columns(array) {
      return Math.max(...array.map(a => Array.isArray(a) ? a.length : 0))
    },
    classname(elem) {
      return elem[0].replace(/@/, '').split('-').join(' ')
    },
    applyStyle() {
      const old = document.getElementById('inserted-style')
      old && old.parentNode.removeChild(old)
      const obj = document.createElement('style')
      obj.setAttribute('id', 'inserted-style')
      obj.appendChild(document.createTextNode(this.style))
      document.getElementsByTagName('head')[0].appendChild(obj)
    },
    isValidMarkId(id) {
      return validMarks.findIndex(e => e === id) !== -1
    }
  },
  mounted() {
    this.applyStyle()
  }
}
</script>
<style lang='scss'>
#viewarea {
  width: 45%;
  float: left;
  #view {
    border: 1px solid gray;
  }
}
#editor {
  width: 45%;
  float: left;
  textarea {
    width: 24%;
    height: 40%;
  }
}
</style>