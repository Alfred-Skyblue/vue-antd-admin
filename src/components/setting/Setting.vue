<template>
  <div class="side-setting">
    <setting-item :title="$t('theme.title')">
      <img-checkbox-group
        @change="values => setTheme({...theme, mode: values[0]})"
        :default-values="[theme.mode]"
      >
        <img-checkbox :title="$t('theme.dark')" img="https://gw.alipayobjects.com/zos/rmsportal/LCkqqYNmvBEbokSDscrm.svg" value="dark"/>
        <img-checkbox :title="$t('theme.light')" img="https://gw.alipayobjects.com/zos/rmsportal/jpRkZQMyYRryryPNtyIC.svg" value="light"/>
        <img-checkbox :title="$t('theme.night')" img="https://gw.alipayobjects.com/zos/antfincdn/hmKaLQvmY2/LCkqqYNmvBEbokSDscrm.svg" value="night"/>
      </img-checkbox-group>
    </setting-item>
    <setting-item :title="$t('theme.color')">
      <color-checkbox-group
        @change="(values, colors) => setTheme({...theme, color: colors[0]})"
        :defaultValues="[palettes.indexOf(theme.color)]" :multiple="false"
      >
        <color-checkbox v-for="(color, index) in palettes" :key="index" :color="color" :value="index" />
      </color-checkbox-group>
    </setting-item>
    <a-divider/>
    <setting-item :title="$t('navigate.title')">
      <img-checkbox-group
        @change="values => setLayout(values[0])"
        :default-values="[layout]"
      >
        <img-checkbox :title="$t('navigate.side')" img="https://gw.alipayobjects.com/zos/rmsportal/JopDzEhOqwOjeNTXkoje.svg" value="side"/>
        <img-checkbox :title="$t('navigate.head')" img="https://gw.alipayobjects.com/zos/rmsportal/KDNDBbriJhLwuqMoxcAr.svg" value="head"/>
        <img-checkbox :title="$t('navigate.mix')" img="https://gw.alipayobjects.com/zos/antfincdn/x8Ob%26B8cy8/LCkqqYNmvBEbokSDscrm.svg" value="mix"/>
      </img-checkbox-group>
    </setting-item>
    <setting-item>
      <a-list :split="false">
        <a-list-item>
          {{$t('navigate.content.title')}}
          <a-select
            :getPopupContainer="getPopupContainer"
            :value="pageWidth"
            @change="setPageWidth"
            class="select-item" size="small" slot="actions"
          >
            <a-select-option value="fluid">{{$t('navigate.content.fluid')}}</a-select-option>
            <a-select-option value="fixed">{{$t('navigate.content.fixed')}}</a-select-option>
          </a-select>
        </a-list-item>
        <a-list-item>
          {{$t('navigate.fixedHeader')}}
          <a-switch :checked="fixedHeader" slot="actions" size="small" @change="setFixedHeader" />
        </a-list-item>
        <a-list-item>
          {{$t('navigate.fixedSideBar')}}
          <a-switch :checked="fixedSideBar" slot="actions" size="small" @change="setFixedSideBar" />
        </a-list-item>
      </a-list>
    </setting-item>
    <a-divider />
    <setting-item :title="$t('other.title')">
      <a-list :split="false">
        <a-list-item>
          {{$t('other.weekMode')}}
          <a-switch :checked="weekMode" slot="actions" size="small" @change="setWeekMode" />
        </a-list-item>
        <a-list-item>
          {{$t('other.multiPages')}}
          <a-switch :checked="multiPage" slot="actions" size="small" @change="setMultiPage" />
        </a-list-item>
        <a-list-item>
          {{$t('other.hideSetting')}}
          <a-switch :checked="hideSetting" slot="actions" size="small" @change="setHideSetting" />
        </a-list-item>
      </a-list>
    </setting-item>
    <a-divider />
    <setting-item :title="$t('animate.title')">
      <a-list :split="false">
        <a-list-item>
          {{$t('animate.disable')}}
          <a-switch :checked="animate.disabled" slot="actions" size="small" @change="val => setAnimate({...animate, disabled: val})" />
        </a-list-item>
        <a-list-item>
          {{$t('animate.effect')}}
          <a-select
            :value="animate.name"
            :getPopupContainer="getPopupContainer"
            @change="val => setAnimate({...animate, name: val})"
            class="select-item" size="small" slot="actions"
          >
            <a-select-option :key="index" :value="item.name" v-for="(item, index) in animates">{{item.alias}}</a-select-option>
          </a-select>
        </a-list-item>
        <a-list-item>
          {{$t('animate.direction')}}
          <a-select
            :value="animate.direction"
            :getPopupContainer="getPopupContainer"
            @change="val => setAnimate({...animate, direction: val})"
            class="select-item" size="small" slot="actions"
          >
            <a-select-option :key="index" :value="item" v-for="(item, index) in directions">{{item}}</a-select-option>
          </a-select>
        </a-list-item>
      </a-list>
    </setting-item>
    <a-alert
      style="max-width: 240px; margin: -16px 0 8px; word-break: break-all"
      type="warning"
      :message="$t('alert')"
    >
    </a-alert>
    <a-button id="copyBtn" :data-clipboard-text="copyConfig" @click="copyCode" style="width: 100%" icon="copy" >{{$t('copy')}}</a-button>
  </div>
</template>

<script>
import SettingItem from './SettingItem'
import {ColorCheckbox, ImgCheckbox} from '@/components/checkbox'
import Clipboard from 'clipboard'
import { mapState, mapMutations } from 'vuex'
import {formatConfig} from '@/utils/formatter'
import {setting} from '@/config/default'
import fastEqual from 'fast-deep-equal'

const ColorCheckboxGroup = ColorCheckbox.Group
const ImgCheckboxGroup = ImgCheckbox.Group
export default {
  name: 'Setting',
  i18n: require('./i18n'),
  components: {ImgCheckboxGroup, ImgCheckbox, ColorCheckboxGroup, ColorCheckbox, SettingItem},
  data() {
    return {
      copyConfig: 'Sorry, you have copied nothing O(∩_∩)O~'
    }
  },
  computed: {
    directions() {
      return this.animates.find(item => item.name == this.animate.name).directions
    },
    ...mapState('setting', ['theme', 'layout', 'animate', 'animates', 'palettes', 'multiPage', 'weekMode', 'fixedHeader', 'fixedSideBar', 'hideSetting', 'pageWidth'])
  },
  watch: {
    'animate.name': function(val) {
      this.setAnimate({name: val, direction: this.directions[0]})
    }
  },
  methods: {
    getPopupContainer() {
      return this.$el.parentNode
    },
    copyCode () {
      let config = {}
      // 提取配置
      let mySetting = this.$store.state.setting
      Object.keys(mySetting).forEach(key => {
        const dftValue = setting[key], myValue = mySetting[key]
        // 只提取与默认配置不同的配置项
        if (dftValue != undefined && !fastEqual(dftValue, myValue)) {
          config[key] = myValue
        }
      })
      this.copyConfig = '// 自定义配置，参考 ./default/setting.config.js，需要自定义的属性在这里配置即可\n'
      this.copyConfig += 'module.exports = '
      this.copyConfig += formatConfig(config)

      let clipboard = new Clipboard('#copyBtn')
      const _this = this
      clipboard.on('success', function () {
        _this.$message.success(`复制成功，覆盖文件 src/config/config.js 然后重启项目即可生效`)
        clipboard.destroy()
      })
    },
    ...mapMutations('setting', ['setTheme', 'setLayout', 'setMultiPage', 'setWeekMode',
      'setFixedSideBar', 'setFixedHeader', 'setAnimate', 'setHideSetting', 'setPageWidth'])
  }
}
</script>

<style lang="less" scoped>
  .side-setting{
    min-height: 100%;
    background-color: @base-bg-color;
    padding: 24px;
    font-size: 14px;
    line-height: 1.5;
    word-wrap: break-word;
    position: relative;
    .flex{
      display: flex;
    }
    .select-item{
      width: 80px;
    }
  }
</style>
