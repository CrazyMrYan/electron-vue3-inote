<template>
  <div
    ref="editor"
    class="editor-layout module-editor empty-content"
    :class="className"
    contenteditable
    v-html="editorContent"
    spellcheck="false"
    placeholder="记笔记..."
    @paste.prevent="paste"
    @input="changeEditorContent"
  ></div>
  <section class="bottom-editor-tools">
    <template v-for="item in bottomIcons" :key="item.name">
      <button class="icon" :title="item.title" @click="editorIconHandle($event, item.name)">
        <i class="iconfont" :class="item.icon"></i>
      </button>
    </template>
  </section>
  <div ref="test"></div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, Ref, watch } from 'vue';
import { debounce } from '@/utils';
import { editorIcons } from '@/config';
import { exeConfig } from '@/store/exeConfig.state';

export default defineComponent({
  props: {
    value: String,
    content: String,
    className: String
  },
  emits: ['on-input', 'update:value'],
  setup(props, { emit }) {
    let editor: Ref<HTMLDivElement | null> = ref(null);
    const bottomIcons = editorIcons;
    const editorContent: Ref<string | undefined> = ref('');

    watch(props, nv => {
      if (!editorContent.value) {
        editorContent.value = nv.content;
      }
    });

    onMounted(() => {
      focus();
    });

    const focus = () => {
      const range = document.createRange();
      range.selectNodeContents(editor.value as HTMLDivElement);
      range.collapse(false);
      const selecton = window.getSelection() as Selection;
      selecton.removeAllRanges();
      selecton.addRange(range);
    };

    const editorIconHandle = (e: Event, name: string) => {
      e.preventDefault();
      document.execCommand(name, false);
    };

    const changeEditorContent = debounce((e: InputEvent) => {
      const editorHtml = (e.target as Element).innerHTML;
      emit('on-input', editorHtml);
    }, exeConfig.syncDelay);

    const paste = (e: ClipboardEvent) => {
      const pasteText = e.clipboardData?.getData('text/plain');
      document.execCommand('insertText', false, pasteText);
    };

    return {
      editor,
      editorIconHandle,
      bottomIcons,
      changeEditorContent,
      paste,
      editorContent
    };
  }
});
</script>

<style lang="less" scoped>
.editor-layout {
  width: 100%;
  height: 100%;
  padding: 12px;
  box-sizing: border-box;
  outline: none;
  font-size: 14px;
  line-height: 1.8;
  overflow-y: auto;
  overflow-x: hidden;
  * {
    font-size: 14px;
    line-height: 1.8;
    word-break: break-all;
  }
  div {
    max-width: 100%;
    overflow-x: auto;
    word-break: break-all;
  }
  span {
    max-width: 100%;
    word-break: break-all;
  }
}

.bottom-editor-tools {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  height: @iconSize;
  background-color: transparent;
  border-top: 1px solid rgba(0, 0, 0, 0.03);
}

.black-content::before {
  color: @gray-color;
}
</style>
