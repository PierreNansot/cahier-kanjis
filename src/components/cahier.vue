<template lang="pug">
div#cahier
  div#left-panel
    div#header
      span Cahier Kanjis
    div#form
      div.list-title
        span Paramètres du cahier
      div.options
          div
            radio(name="font",
                  value="default",
                  v-model="font") Police standard
            radio(name="font",
                  value="hand",
                  v-model="font") Manuscrite
            radio(name="font",
                  value="stroke",
                  v-model="font") Aide au tracé
          div
            checkbox(name="translation",
                     value="true",
                     v-model="translation") Sens et lecture
            checkbox(name="vertical",
                     value="false",
                     v-model="vertical") Verticale
            checkbox(name="double-line",
                     value="true",
                     v-model="doubleLine") Deux lignes
          div
            radio(name="help",
                  value="kanji-help",
                  v-model="help") Kanji & aides
            radio(name="help",
                  value="kanji-only",
                  v-model="help") Kanji
            radio(name="help",
                  value="help-only",
                  v-model="help") Aide
    ul
      li(v-for="(grade, index) in gradeTitle")
        div.list-title
          span {{ grade }}
          div.select(@click="toogleGradeKanjis(index + 1, 1)") Tous
          div.select(@click="toogleGradeKanjis(index + 1, 0)") Aucun
        div.list-kanji
          div(v-for="k in getGradeKanjis(index + 1)",
              :key="k['character']",
              :class="{ active: isSelected(k.character) }",
              @click="toogleKanji(k.character)",
              v-tooltip="getTooltipContent(k)") {{ k['character'] }}
  div#content
    div#cahier-header
      span Aperçu
      button(v-if="selected.length > 0", @click="print") ⎙ Imprimer
    div#cahier-example
      div.kanjis-selected
        div.kanji-cases(v-for="(k, index) in getExampleKanjis",
                        :key="k.id",
                        :class="{vertical: vertical}")
          div.meaning(v-for="(m, index) in k['meaning'].slice(0,2)", v-if="translation")
            strong Signification :
            span {{ m['translation_fr'] }}
            strong Onyomi :
            span {{ m['onyomi'] }}
            strong Kunyomi :
            span {{ m['kunyomi'] }}
          div.cases(:class="{ stroke: font === 'stroke', hand: font === 'hand' }")
            div.first-case
              span {{ k['character'] }}
            div.grey-case(v-if="help === 'kanji-help'") {{ k['character'] }}
            div(v-else)
            div.grey-case(v-if="help === 'kanji-help'") {{ k['character'] }}
            div(v-else)
            div(v-for="n in getMaxCases - 3")
          div.cases(v-if="doubleLine")
            div(v-for="n in getMaxCases")
  div#cahier-print
    div.kanjis-selected
      div.kanji-cases(v-for="k in getSelectedKanjis",
                      :key="k.id",
                      :class="{vertical: vertical}")
        div.meaning(v-for="(m, index) in k['meaning'].slice(0,2)", v-if="translation")
          strong Signification :
          span {{ m['translation_fr'] }}
          strong Onyomi :
          span {{ m['onyomi'] }}
          strong Kunyomi :
          span {{ m['kunyomi'] }}
        div.cases(:class="{ stroke: font === 'stroke', hand: font === 'hand' }")
          div.first-case()
            span {{ k['character'] }}
          div.grey-case(v-if="help === 'kanji-help'") {{ k['character'] }}
          div(v-else)
          div.grey-case(v-if="help === 'kanji-help'") {{ k['character'] }}
          div(v-else)
          div(v-for="n in getMaxCases - 3")
        div.cases(v-if="doubleLine")
          div(v-for="n in getMaxCases")


</template>

<script>
  import Kanji from './../../kanji/kanji.json'
  import _indexOf from 'lodash/indexOf'
  import _filter from 'lodash/filter'
  import _find from 'lodash/find'

  export default {
    name: 'cahier',
    data () {
      return {
        kanji: Kanji,
        font: "default",
        doubleLine: true,
        translation: true,
        vertical: false,
        help: 'kanji-help',
        selected: [],
        gradeTitle: [
          'Niveau 1',
          'Niveau 2',
          'Niveau 3',
          'Niveau 4',
          'Niveau 5',
          'Niveau 6',
          'Niveau 7',
          'Jinmei',
          'Rare'
        ]
      }
    },
    watch: {
      vertical: function (newVertical) {
        if (newVertical) {
          this.translation = false
        }
      },
      translation: function (newTranslation) {
        if (newTranslation) {
          this.vertical = false
        }
      }
    },
    computed: {
      getSelectedKanjis: function () {
        let selectedKanjis = []
        this.selected.forEach(function(ks) {
          selectedKanjis.push(_find(this.kanji, function(k) {return k['character'] === ks}))
        }, this)
        return selectedKanjis
      },
      getExampleKanjis: function () {
        let selectedKanjis = []
        this.selected.forEach(function(ks) {
          selectedKanjis.push(_find(this.kanji, function(k) {return k['character'] === ks}))
        }, this)

        let max = 6
        if (!this.translation && this.doubleLine && !this.vertical) {
          max = 7
        } else if (!this.translation && !this.doubleLine && !this.vertical) {
          max = 14
        } else if (this.translation && !this.doubleLine && !this.vertical) {
          max = 10
        } else if (!this.translation && !this.doubleLine && this.vertical) {
          max = 10
        } else if (!this.translation && this.doubleLine && this.vertical) {
          max = 5
        }


        return selectedKanjis.slice(0,max)
      },
      getMaxCases: function () {
        if(!this.vertical) {
          return 10
        } else {
          return 13
        }
      }
    },
    methods: {
      toogleGradeKanjis: function (idx, select) {
        this.getGradeKanjis(idx).forEach(function(k) {
          let kanjiIndex = _indexOf(this.selected, k['character'])
          if (kanjiIndex === -1 && select) {
            this.selected.push(k['character'])
          } else if (kanjiIndex !== -1 && !select) {
            this.selected.splice(kanjiIndex, 1)
          }
        }, this)
      },
      toogleKanji: function (k) {
        let kanjiIndex = _indexOf(this.selected, k)
        if (kanjiIndex !== -1) {
          this.selected.splice(kanjiIndex, 1)
        } else {
          this.selected.push(k)
        }
      },
      isSelected: function (k) {
        return _indexOf(this.selected, k) !== -1
      },
      getTooltipContent: function (k) {
        return this.getMeaningFormatted(k['meaning'][0])
      },
      getGradeKanjis: function (idx) {
        return _filter(this.kanji, function(k) {return k['grade'] === 'grade' + idx})
      },
      getMeaningFormatted: function (m) {
        let meaning = ''
        if(m['kunyomi'] !== undefined) {
          meaning = meaning + m['kunyomi']
        }
        if(m['onyomi'] !== undefined) {
          meaning = meaning + m['onyomi']
        }
        return meaning + ' (' + m['translation_fr'] + ')'
      },
      print: function () {
        window.print()
      }
    }
  }
</script>

<style lang="sass">
  @import "../sass/cahier.sass"

  html, body, #app, #cahier
    height: 100%
    margin: 0
    overflow: hidden
    background-color: rgba(255,255,255,0.8)
    ul
      padding: 0
      list-style-type: none
    #content
      min-width: 800px
      max-width: 70%
      height: 100%
      display: flex
      flex-direction: column
      align-items: center
      overflow: hidden
      overflow-y: scroll
      padding: 12px 0
      #cahier-header
        width: 210mm
        display: flex
        justify-content: space-between
        align-items: center
        min-height: 60px
        span
          color: #323232
          font-size: 42px
        button
          background-color: #E34C26
          height: 32px
          border: none
          color: white
          font-size: 20px
          padding: 0 8px
          &:hover
            cursor: pointer
            border-bottom: solid 4px rgba(0,0,0,0.3)
      #cahier-example
        width: 190mm
        height: 297mm
        padding: 15mm 20mm
        margin-bottom: 24px
        background-color: white
        box-shadow: 0 6px 12px rgba(0,0,0,0.16), 0 6px 12px rgba(0,0,0,0.23)
        &>.kanjis-selected>.kanji-cases>.cases
          &.stroke
            font-family: 'kanjiStroke'
            font-size: 64px
          &.hand
            font-size: 64px
            font-family: 'hand'


    #cahier-print
      visibility: hidden
      .kanjis-selected
        page-break-inside: avoid
        .kanji-cases
          page-break-inside: avoid
          .meaning
            font-size: 16px
          .cases
            page-break-inside: avoid
            font-size: 90px
            &>div
              page-break-inside: avoid
              min-height: 96px
              min-width: 96px
          &.vertical
            &>.cases
              page-break-inside: avoid
              font-size: 72px
              &>div
                min-height: 62px
                min-width: 62px
    </style>
