<template>
  <div class="editor">
    <editor-menu-bubble
      :editor="editor"
      :keep-in-bounds="keepInBounds"
      v-slot="{ commands, isActive, menu }"
    >
      <div
        class="menububble"
        :class="{ 'is-active': menu.isActive }"
        :style="`left: ${menu.left}px; bottom: ${menu.bottom}px;`"
      >
        <button
          class="menububble__button"
          :class="{ 'is-active': isActive.bold() }"
          @click="commands.bold"
        >
          bold
        </button>

        <button
          class="menububble__button"
          :class="{ 'is-active': isActive.italic() }"
          @click="commands.italic"
        >
          i
        </button>

        <button
          class="menububble__button"
          :class="{ 'is-active': isActive.code() }"
          @click="commands.code"
        >
          code
        </button>
      </div>
    </editor-menu-bubble>

    <editor-content class="editor__content" :editor="editor" />

    <button @click="output = editor.getHTML()">
      get HTML
    </button>

    <div>
      <textarea style="width: 100%;" readonly :value="output" />
    </div>
  </div>
</template>

<script>
import { Editor, EditorContent, EditorMenuBubble, Extension } from "tiptap";
import { Slice, Fragment, Node } from "prosemirror-model";
import { HardBreak, Bold, Code, Italic, History } from "tiptap-extensions";

function clipboardTextParser(text, context) {
  // paste content as plain text
  text = text.replace(/<("[^"]*"|'[^']*'|[^'">])*>/g,'');
  const blocks = text.replace().split(/(?:\r\n?|\n)/);
  const nodes = [];

  blocks.forEach((line, i, arr) => {
    const scheme = context.doc.type.schema;
    if (line.length > 0) {
      nodes.push(Node.fromJSON(scheme, { type: "text", text: line }));
    }
    if ((i + 1) !== arr.length) {
      nodes.push(Node.fromJSON(scheme, { type: "hard_break" }));
    }
  });

  const fragment = Fragment.fromArray(nodes);
  return Slice.maxOpen(fragment);
}

const CustomeEnter = class extends Extension {
  keys() {
    return {
      Enter(state, dispatch, view) {
        const { schema, tr } = view.state;

        const hard_break = schema.nodes.hard_break;
        const transaction = tr
          .replaceSelectionWith(hard_break.create())
          .scrollIntoView();
        view.dispatch(transaction);
        return true;
      },
    };
  }
};

export default {
  components: {
    EditorContent,
    EditorMenuBubble,
  },
  data() {
    return {
      output: "",
      keepInBounds: true,
      editor: new Editor({
        extensions: [
          new HardBreak(),
          new Bold(),
          new Code(),
          new Italic(),
          new History(),
          new CustomeEnter(),
        ],
        content: `
          Hey, try to select some text here. There will popup a menu for selecting some inline styles. <em>Remember:</em> you have full control about content and styling of this menu.
        `,
        editorProps: {
          clipboardTextParser,
        },
      }),
    };
  },
  beforeDestroy() {
    this.editor.destroy();
  },
};
</script>

<style lang="scss">
.editor {
  position: relative;
  max-width: 30rem;
  margin: 0 auto 5rem auto;

  &__content {
    overflow-wrap: break-word;
    word-wrap: break-word;
    word-break: break-word;

    * {
      caret-color: currentColor;
    }

    p {
      margin: 0;
    }

    pre {
      padding: 0.7rem 1rem;
      border-radius: 5px;
      background: #000;
      color: #fff;
      font-size: 0.8rem;
      overflow-x: auto;

      code {
        display: block;
      }
    }

    p code {
      padding: 0.2rem 0.4rem;
      border-radius: 5px;
      font-size: 0.8rem;
      font-weight: bold;
      background: rgba(#000, 0.1);
      color: rgba(#000, 0.8);
    }

    ul,
    ol {
      padding-left: 1rem;
    }

    li > p,
    li > ol,
    li > ul {
      margin: 0;
    }

    a {
      color: inherit;
    }

    blockquote {
      border-left: 3px solid rgba(#000, 0.1);
      color: rgba(#000, 0.8);
      padding-left: 0.8rem;
      font-style: italic;

      p {
        margin: 0;
      }
    }

    img {
      max-width: 100%;
      border-radius: 3px;
    }

    table {
      border-collapse: collapse;
      table-layout: fixed;
      width: 100%;
      margin: 0;
      overflow: hidden;

      td,
      th {
        min-width: 1em;
        border: 2px solid #ddd;
        padding: 3px 5px;
        vertical-align: top;
        box-sizing: border-box;
        position: relative;
        > * {
          margin-bottom: 0;
        }
      }

      th {
        font-weight: bold;
        text-align: left;
      }

      .selectedCell:after {
        z-index: 2;
        position: absolute;
        content: "";
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        background: rgba(200, 200, 255, 0.4);
        pointer-events: none;
      }

      .column-resize-handle {
        position: absolute;
        right: -2px;
        top: 0;
        bottom: 0;
        width: 4px;
        z-index: 20;
        background-color: #adf;
        pointer-events: none;
      }
    }

    .tableWrapper {
      margin: 1em 0;
      overflow-x: auto;
    }

    .resize-cursor {
      cursor: ew-resize;
      cursor: col-resize;
    }
  }
}

.menububble {
  position: absolute;
  display: flex;
  z-index: 20;
  background: #000;
  border-radius: 5px;
  padding: 0.3rem;
  margin-bottom: 0.5rem;
  transform: translateX(-50%);
  visibility: hidden;
  opacity: 0;
  transition: opacity 0.2s, visibility 0.2s;

  &.is-active {
    opacity: 1;
    visibility: visible;
  }

  &__button {
    display: inline-flex;
    background: transparent;
    border: 0;
    color: #fff;
    padding: 0.2rem 0.5rem;
    margin-right: 0.2rem;
    border-radius: 3px;
    cursor: pointer;

    &:last-child {
      margin-right: 0;
    }

    &:hover {
      background-color: rgba(#fff, 0.1);
    }

    &.is-active {
      background-color: rgba(#fff, 0.2);
    }
  }

  &__form {
    display: flex;
    align-items: center;
  }

  &__input {
    font: inherit;
    border: none;
    background: transparent;
    color: #fff;
  }
}
</style>
