<template>
  <m-list-construction :title="$t('Create File')">
    <template slot="content">
      <div class="resource-create-model">
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('File Name')}}</template>
          <template slot="content">
            <x-input
                    type="input"
                    v-model="fileName"
                    style="width: 300px;"
                    :placeholder="$t('Please enter name')"
                    autocomplete="off">
            </x-input>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('File Format')}}</template>
          <template slot="content">
            <x-select v-model="suffix" style="width: 100px;" @on-change="_onChange">
              <x-option
                      v-for="city in fileTypeList"
                      :key="city"
                      :value="city"
                      :label="city">
              </x-option>
            </x-select>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name">{{$t('Description')}}</template>
          <template slot="content">
            <x-input
                    type="textarea"
                    v-model="desc"
                    style="width: 430px;"
                    :placeholder="$t('Please enter description')"
                    autocomplete="off">
            </x-input>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name"><b>*</b>{{$t('File Content')}}</template>
          <template slot="content">
            <textarea id="code-create-mirror" name="code-create-mirror"></textarea>
          </template>
        </m-list-box-f>
        <m-list-box-f>
          <template slot="name">&nbsp;</template>
          <template slot="content">
            <div class="submit">
              <x-button type="primary" shape="circle" :loading="spinnerLoading" @click="ok()" v-ps="['GENERAL_USER']">{{spinnerLoading ? 'Loading...' : $t('Create')}} </x-button>
              <x-button type="text" @click="() => $router.push({name: 'file'})"> {{$t('Cancel')}} </x-button>
            </div>
          </template>
        </m-list-box-f>
      </div>
    </template>
  </m-list-construction>
</template>
<script>
  import i18n from '@/module/i18n'
  import { mapActions } from 'vuex'
  import { filtTypeArr } from '../_source/common'
  import { handlerSuffix } from '../details/_source/utils'
  import codemirror from '../_source/codemirror'
  import mListBoxF from '@/module/components/listBoxF/listBoxF'
  import mSpin from '@/module/components/spin/spin'
  import mConditions from '@/module/components/conditions/conditions'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  let editor
  export default {
    name: 'resource-list-create-FILE',
    data () {
      return {
        suffix: 'sh',
        fileName: '',
        desc: '',
        fileTypeList: filtTypeArr,
        content: '',
        spinnerLoading: false
      }
    },
    props: {},
    methods: {
      ...mapActions('resource', ['createResourceFile']),
      ok () {
        if (this._validation()) {
          this.spinnerLoading = true
          this.createResourceFile({
            type: 'FILE',
            fileName: this.fileName,
            suffix: this.suffix,
            desc: this.desc,
            content: editor.getValue()
          }).then(res => {
            this.$message.success(res.msg)
            setTimeout(() => {
              this.spinnerLoading = false
              this.$router.push({ name: 'file' })
            }, 800)
          }).catch(e => {
            this.$message.error(e.msg || '')
            this.spinnerLoading = false
          })
        }
      },
      _validation () {
        if (!this.fileName) {
          this.$message.warning(`${i18n.$t('Please enter resource name')}`)
          return false
        }
        if (!editor.getValue()) {
          this.$message.warning(`${i18n.$t('Please enter the resource content')}`)
          return false
        }

        return true
      },
      /**
       * Processing code highlighting
       */
      _handlerEditor () {
        // editor
        editor = codemirror('code-create-mirror', {
          mode: 'shell',
          readOnly: false
        })

        this.keypress = () => {
          if (!editor.getOption('readOnly')) {
            editor.showHint({
              completeSingle: false
            })
          }
        }

        // Monitor keyboard
        editor.on('keypress', this.keypress)
      },
      _onChange (val) {
        editor.setOption('mode', handlerSuffix['.' + val.label])
      }
    },
    watch: {},
    created () {
    },
    mounted () {
      this._handlerEditor()
    },
    destroyed () {
      editor.toTextArea() // 卸载
      editor.off($('.code-create-mirror'), 'keypress', this.keypress)
    },
    computed: {},
    components: { mListConstruction, mConditions, mSpin, mListBoxF }
  }
</script>

<style lang="scss" rel="stylesheet/scss">
  .resource-create-model {
    padding: 30px;
  }
  .CodeMirror {
    border:1px solid #DDDEDD;
    border-radius: 3px;
  }
</style>
