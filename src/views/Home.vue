<template>
  <div class="mdui-row mdui-typo mdui-center" style="max-width: 1012px;">
    <div class="mdui-col-xs-12">
      <h1 class="mdui-m-t-0">{{$t('app.title')}}</h1>
      <p>目前支持国服、国际服、日服、韩服，选语言即可切换</p>
      <p>Github: <a href="https://github.com/Tsuk1ko/arknights-toolbox" target="_blank">Tsuk1ko/arknights-toolbox</a></p>
      <p>宗旨是简洁美观且对移动设备友好，以及 Material Design 天下第一（。）</p>
      <p>如果有好的想法、建议、希望增加的功能，或者发现了 bug，欢迎到项目中提 <a href="https://github.com/Tsuk1ko/arknights-toolbox/issues" target="_blank">issue</a> 或提交 pr</p>
      <h2>{{$t('common.setting')}}</h2>
      <locale-select :key="$root.localeSelectKey" />
      <p>
        <mdui-switch v-model="setting.rememberLastPage">{{$t('home.setting.rememberLastPage')}}</mdui-switch>
        <mdui-switch v-if="canUseCDN" v-model="setting.imageCDN" :mdui-tooltip="`{content:'${$t('home.setting.imageCDNTip')}',position:'top'}`">{{$t('home.setting.imageCDN')}}</mdui-switch>
      </p>
      <p class="no-sl">
        <button class="mdui-btn mdui-btn-raised mdui-ripple mdui-color-pink-accent mdui-m-r-1" @click="clearStorage">{{$t('home.setting.clearStorage')}}</button><i class="mdui-icon material-icons mdui-m-r-1 help no-sl" :mdui-tooltip="`{content:'${$t('home.setting.clearStorageTip')}',position:'top'}`">info_outline</i>{{$t('home.used')}}{{lsSize}}
      </p>
      <p class="no-sl">
        <button class="mdui-btn mdui-btn-raised mdui-ripple mdui-color-pink-accent mdui-m-r-1" :disabled="!checkNavigatorStorage()" @click="clearCaches">{{$t('home.setting.clearCaches')}}</button><i class="mdui-icon material-icons mdui-m-r-1 help no-sl" :mdui-tooltip="`{content:'${$t('home.setting.clearCachesTip')}',position:'top'}`">info_outline</i>{{$t('home.used')}}{{csSize}}
      </p>
      <add-to-home-screen />
      <h2>主要功能</h2>
      <ul>
        <li>公开招募计算 + 词条截图识别</li>
        <li>精英材料计算 + 导入干员材料预设 + 素材获取最优策略规划</li>
        <li>干员升级计算</li>
        <li>基建技能筛选</li>
      </ul>
      <p>※ 如果公招词条识别出现遗漏现象且您愿意帮助我改进结果，请到 Console 复制 OCR 输出，并截图当前识别结果，到 GitHub 上提交 issue</p>
      <h2>数据素材来源及鸣谢</h2>
      <ul>
        <li><a href="https://github.com/Kengxxiao/ArknightsGameData" target="_blank">Kengxxiao/ArknightsGameData</a>（数据）</li>
        <li><a href="http://ak.mooncell.wiki" target="_blank">PRTS Wiki</a>（干员头像、跳转至干员详情）（很棒的 wiki，大家可以多支持一下）</li>
        <li><a href="https://github.com/graueneko/graueneko.github.io" target="_blank">一只灰猫</a>（干员升级数据）</li>
        <li><a href="https://github.com/Houdou/arkgraph" target="_blank">ark-nights.com</a>（材料图片）</li>
        <li><a href="https://penguin-stats.io/" target="_blank">企鹅物流数据统计</a>（掉落数据）</li>
        <li><a href="https://bbs.nga.cn/read.php?tid=17507710" target="_blank">素材获取最优策略规划</a>（思路）</li>
      </ul>
      <h2>{{$t('home.changelog')}}</h2>
      <changelog />
    </div>
  </div>
</template>

<script>
import Changelog from '../components/Changelog';
import AddToHomeScreen from '../components/AddToHomeScreen';
import LocaleSelect from '../components/LocaleSelect';
import _ from 'lodash';
import utf8BufferSize from 'utf8-buffer-size';

export default {
  name: 'home',
  components: {
    Changelog,
    AddToHomeScreen,
    LocaleSelect,
  },
  data() {
    return {
      lsSize: this.calcLsSize(),
      csSize: this.$t('home.calculating'),
      setting: this.$root.setting,
      canUseCDN: !!process.env.VUE_APP_REPOSITORY,
    };
  },
  methods: {
    checkNavigatorStorage: () => 'storage' in navigator && 'estimate' in navigator.storage,
    clearStorage() {
      localStorage.clear();
      this.$snackbar(this.$t('common.success'));
      this.lsSize = this.calcLsSize();
    },
    async clearCaches() {
      if (!('serviceWorker' in navigator)) return;
      const cacheKeys = (await caches.keys()).filter(key => _.includes(key, 'runtime'));
      const cacheList = await Promise.all(cacheKeys.map(key => caches.open(key)));
      for (const cache of cacheList) {
        await cache.keys().then(keys => Promise.all(keys.map(key => cache.delete(key))));
      }
      await Promise.all(cacheKeys.map(key => caches.delete(key)));
      this.$snackbar(this.$t('common.success'));
      this.csSize = await this.calcCsSize();
    },
    calcLsSize() {
      return this.$root.calcSize(_.sumBy(Object.values(localStorage), utf8BufferSize) * 2);
    },
    calcCsSize() {
      if (!this.checkNavigatorStorage()) return Promise.resolve('N/A');
      return navigator.storage
        .estimate()
        .then(({ usage }) => this.$root.calcSize(usage))
        .catch(() => 'N/A');
    },
  },
  created() {
    this.calcCsSize().then(size => (this.csSize = size));
  },
};
</script>
