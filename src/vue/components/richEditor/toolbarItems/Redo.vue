<template>
    <div class="toolbar-item" ref="toolbarItem" :class="{ active: active}">
        <i-button border type="icon" @click="handleClick" icon="redo"></i-button>
    </div>
</template>

<script>
    export default {
        name: 'Redo',
        props: {
            editor: {
                type: Object
            }
        },
        data () {
          return {
              active: false
          }
        },
        methods: {
            handleClick (e) {
                const editor = this.editor;
                // 执行 bold 命令
                editor.command.do('redo');
            }
        }
    }
</script>

<style scoped>

</style>