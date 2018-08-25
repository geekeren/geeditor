<template>
  <div class='markdown-container'>
    <div class='markdown-input-section section'>
      <header>
        <span class="title menu_item">极客MD编辑器</span>
      </header>
      <textarea class='edit-text-area content'
                v-model='rawInputMd'
                @input='onMdInput'>
      </textarea>
    </div>
    <div class='html-preview-section section'>
      <header>
        <button class="menu_item" ref="button">复制</button>
      </header>
      <div class='html-preview-content content mail-content'
           ref="parsedHtmlNode">
        <div class="parsed-html" v-html='parsedHtml'></div>
        <div class="copyright-info">
          本邮件自豪地采用了“<a href="https://md.wangbaiyuan.cn">极客MD</a>”编辑
        </div>
      </div>
    </div>
  </div>
</template>
<script lang='ts'>
  import {Marked, Renderer} from "marked-ts";
  import {Component, Vue} from "vue-property-decorator";
  import {highlightAuto} from "highlight.js";
  import Clipboard from "clipboard";

  Marked.setOptions
  ({
    renderer: new Renderer(),
    gfm: true,
    tables: true,
    breaks: false,
    pedantic: false,
    sanitize: false,
    smartLists: true,
    smartypants: false,
    highlight: (code) => {
      return highlightAuto(code).value;
    },

  });
  @Component
  export default class Markdown extends Vue {
    public rawInputMd: string = "";
    public parsedHtml: string = "";

    mounted() {
      let content: string | null = JSON.parse(localStorage.getItem("md.content"));
      if(!content) {
        this.$http.get("/data/example.md")
          .then(data => {
            this.rawInputMd = data.body;
            this.updateHtmlPreview();
          });
      }
      this.rawInputMd = content;
      this.updateHtmlPreview();
      const parsedHtmlNode: Element = this.$refs.parsedHtmlNode;
      const copyBtn: NodeListOf<Element> = this.$refs.button;
      const clipboard = new Clipboard( copyBtn, {
        target: (elem : any) => parsedHtmlNode
      });
      clipboard.on("success", this.onCopy);
      clipboard.on("error", this.onError);
    }

    public onMdInput(e: any) {
      this.updateHtmlPreview();
      localStorage.setItem("md.content", JSON.stringify(this.rawInputMd));
    }

    protected updateHtmlPreview() {
      const mdToParsed = this.parseRawMd(this.rawInputMd);
      this.parsedHtml = Marked.parse(mdToParsed);
    }

    protected onCopy(e: any) {
      alert("复制成功!");
    }

    protected onError(e: any) {
      alert("复制失败");
    }

    private parseRawMd(rawMd: string) {
      return rawMd.replace("\n", "\r\n\n");
    }
  }
</script>
<style lang="scss">

  .markdown-container {
    width: 100%;
    height: 100%;
    flex: 2;
    display: -webkit-flex;
    display: flex;
    flex-direction: row;
    background: #eee;
    .section {
      position: relative;
      text-align: left;
      -webkit-box-flex: 1;
      -ms-flex: 1;
      flex: 1;
      display: flex;
      flex-direction: column;
      overflow-x: hidden;
      overflow-y: auto;
      margin: 0px;
      border-radius: 2px;
    }
    .parsed-html {
      min-height: 100%;
      margin-top: 3em;
    }
    .content {
      padding: 14px;
      flex: 1;
      overflow: auto;
    }
    .edit-text-area {
      -webkit-appearance: none;
      -moz-appearance: none;
      background-color: transparent;
      border: none;
      border-radius: .4rem;
      box-shadow: none;
      box-sizing: inherit;
      color: #fff;
      width: 100%;
      height: 100%;
      outline: none;
      padding-top: 3.5em;
    }
    .markdown-input-section {
      background: #333;
      header {
        .title {
          color: #20d6ff;
        }
      }
    }
    .html-preview-section {
      background: #fff;
      .copyright-info {
        display: none;
      }
      header {
        text-align: right;
        right: 0px;
        .menu_item {
          background: #1e6bb8;
          color: white;
          padding: 8px 12px;
        }

        .menu_item:hover {
          background: #20d6ff;
          box-shadow: #20d6ff;
          cursor: pointer;
        }
      }
    }
    .mail-content {
      padding: 2px;
      .parsed-html {
        background: #ffffff;
        max-width: 860px;
        margin: 0 auto;
        padding: 3.5em 20px;
        box-shadow: 3px 5px 20px #b8dcec;
      }
      .copyright-info {
        display: block;
        max-width: 860px;
        margin: 0 auto;
        padding: 12px;
        font-size: 12px;
        text-align: right;
        font-style: italic;
      }
    }
  }

</style>