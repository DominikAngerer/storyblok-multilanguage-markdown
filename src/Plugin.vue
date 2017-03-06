<template>
  <div>
    <div style="background-color:#fff;border:1px solid #CCC">
        <button class="uk-button" v-on:click.prevent="wrap(0, '**', '**')"><i class="uk-icon-bold"></i></button><a class="uk-button" href="https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet" target="_blank"><i class="uk-icon-question"></i></a>
        <button class="uk-button" v-on:click.prevent="openOverlay"><i class="uk-icon-expand"></i></button>
        <div class="uk-float-right">
        <a v-for="(index,language) in model.available_languages" v-on:click.prevent="switchLanguage(language)" :class="classCurrentLanguage(language)" class="uk-button uk-text-uppercase" href="#">{{language}}</a>
        </div> 
    </div>
    <textarea v-model="model.markdown" rows="5" class="uk-width-1-1" debounce="300"></textarea>
    <p class="uk-text-small uk-text-muted">Currently editing <span class="uk-text-uppercase">{{current_language}}</span></p>
    <div class="overlay" :class="{'overlay--open': open}">
        <div class="overlay__inner">
            <div class="uk-clearfix">
              <div class="uk-float-right"><a v-on:click="close" class="uk-button"><i class="uk-icon-close"></i> Ready</a></div>
              <h1 class="overlay__headline">Markdown Editor (currently editing <span class="uk-text-uppercase">{{current_language}}</span>)</h1></div>
            <div class="overlay__list">
                <div class="uk-grid">
                    <div class="uk-width-1-2">
                        <div style="background-color:#fff;border:1px solid #CCC">
                           <button class="uk-button" v-on:click.prevent="wrap(1, '**', '**')"><i class="uk-icon-bold"></i></button><a class="uk-button" href="https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet" target="_blank"><i class="uk-icon-question"></i></a>
                           <div class="uk-float-right">
                            <a v-for="(index,language) in model.available_languages" v-on:click.prevent="switchLanguage(language)" :class="classCurrentLanguage(language)" class="uk-button uk-text-uppercase" href="#">{{language}}</a>
                           </div> 
                        </div>
                        <textarea v-model="model.markdown" class="uk-width-1-1" style="height: calc(100vh - 200px)" debounce="300"></textarea>
                    </div>
                    <div class="uk-width-1-2">
                        <div class="uk-text-muted uk-margin-bottom">Output for: <span class="uk-text-uppercase">{{current_language}}</span></div>
                        <div v-html="parsed_markdown"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  mixins: [window.Storyblok.plugin],
  data: function() {
    return {
      open: false,
      parsed_markdown: '',
      current_language: ''
    }
  },
  computed: {
    options() {
      var options = {}
      for (var index = 0; index < this.schema.options.length; index++) {
        var element = this.schema.options[index]
        options[element.name] = element.value;
      }
      return options
    }
  },
  methods: {
    initWith: function() {
      return {
        plugin: 'multilanguage-markdown',
        markdown: '',
        available_languages: [],
        text: {}
      };
    },
    classCurrentLanguage: function(language) {
      if(language == this.current_language) {
        return 'uk-button-primary'
      } else {
        return ''
      }
    },
    initEditor: function() {
      if(this.model.text[this.current_language]) {
        this.model.markdown = this.model.text[this.current_language];
      } else {
        this.model.markdown = ''
      }
      this.parsed_markdown = marked(this.model.markdown)
    },
    close: function() {
      this.open = false;
    },
    openOverlay: function() {
      this.open = true;
    },
    switchLanguage: function(language) {
      this.current_language = language
      if(!this.model.text[this.current_language]) {
        this.model.text[this.current_language] = ''
      }
      this.model.markdown = this.model.text[this.current_language];
    },
    wrap: function(el, a, i) {
      var g = this.$el.querySelectorAll('textarea')[el];
      g.focus();
      if (g.setSelectionRange) {
        var c = g.scrollTop;
        var e = g.selectionStart;
        var f = g.selectionEnd;
        g.value = g.value.substring(0, g.selectionStart) + a + g.value.substring(g.selectionStart, g.selectionEnd) + i + g.value.substring(g.selectionEnd, g.value.length);
        g.selectionStart = e;
        g.selectionEnd = f + a.length + i.length;
        g.scrollTop = c;
      } else {
        if (document.selection && document.selection.createRange) {
          g.focus();
          var b = document.selection.createRange();
          if (b.text !== "") {
            b.text = a + b.text + i;
          } else {
            b.text = a + "REPLACE" + i;
          }
          g.focus();
        }
      }
      jQuery(g).trigger('change');
    }
  },
  events: {
    'plugin:created': function() {
      if (!this.schema.options) {
        console.warn('multilanguage-markdown: We guess you\'re working locally so we will set the options for you.');
        this.schema.options = [
          {
            name:'available_languages',
            value:'en,de'
          }
        ]
      }
      if(!this.options.available_languages) {
        console.error('multilanguage-markdown: If you need other languages than EN add them as "available_languages" option as csv (eg. en,de,es)');
        return;
      }
      this.model.available_languages = this.options.available_languages.split(',')
      this.current_language = this.model.available_languages[0];
      jQuery.getScript('https://cdnjs.cloudflare.com/ajax/libs/marked/0.3.2/marked.min.js', this.initEditor.bind(this))
    }
  },
  watch: {
    'model': {
      handler: function (value) {
        this.$emit('changed-model', value);
        if(window.marked) {
          this.parsed_markdown = window.marked(this.model.markdown)
        }
        this.model.text[this.current_language] = this.model.markdown;
      },
      deep: true
    }
  }
}
</script>

<style>
</style>
