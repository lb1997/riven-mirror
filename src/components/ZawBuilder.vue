<template>
  <div class="modularbuilder zaw">
    <el-steps :active="part" finish-status="success">
      <el-step :title="$t('zaw.selectStrike')"></el-step>
      <el-step :title="$t('zaw.selectGrip')"></el-step>
      <el-step :title="$t('zaw.selectLinks')"></el-step>
    </el-steps>
    <!-- 击打部 -->
    <div class="partlist" v-if="part === 0">
      <div class="part-box" v-for="item in strikeList" :key="item.id">
        <el-radio class="part" v-model="strike" :label="item" border>
          <div class="snapshot">
            <img :src="`https://cdn.riven.im/img/zawStrike${item.id.replace(/ /g, '')}.m.png`" :alt="item.id" width="100%">
          </div>
          <div class="name">
            {{$t(`messages.${item.name}`)}}
          </div>
          <div class="type">
            {{$t(`zaw.${item.oneHand.type}`)}} / {{$t(`zaw.${item.twoHand.type}`)}}
          </div>
        </el-radio>
      </div>
    </div>
    <!-- 握柄部 -->
    <div class="partlist" v-if="part === 1">
      <div class="part-box" v-for="item in gripList" :key="item.id">
        <el-radio class="part" v-model="grip" :label="item" border>
          <div class="snapshot">
            <img :src="`https://cdn.riven.im/img/zawGrip${item.id.replace(/ /g, '')}.m.png`" :alt="item.id" width="100%">
          </div>
          <div class="name">
            {{$t(`messages.${item.name}`)}}
          </div>
          <div class="type">
            {{$t(`zaw.${strike[item.twoHand ? 'twoHand' : 'oneHand'].type}`)}}
          </div>
          <div class="prop">
            <span>{{+((60+strike.speed+item.speed)/60).toFixed(3)}} {{$t(`modular.fireRate`)}}</span>
            <span>{{gripDmg(item)}} {{$t(`modular.damage`)}}</span>
          </div>
        </el-radio>
      </div>
    </div>
    <!-- 环接部 -->
    <div class="partlist" v-if="part === 2">
      <div class="part-box" v-for="item in linksList" :key="item.id">
        <el-radio class="part" v-model="links" :label="item" border>
          <div class="snapshot">
            <img :src="`https://cdn.riven.im/img/zawLink${item.id.replace(/II/g, '2').replace(/ /g, '')}.m.png`" :alt="item.id" width="100%">
          </div>
          <div class="name">
            {{$t(`messages.${item.name}`)}}
          </div>
          <div class="prop">
            <span>{{+((60+strike.speed+grip.speed+item.speed)/60).toFixed(3)}} {{$t(`modular.fireRate`)}}</span>
            <span>{{linkDmg(item)}} {{$t(`modular.damage`)}}</span>
            <span>{{10+strike.crit+item.crit}}% {{$t(`modular.critChance`)}}</span>
            <span>{{10+strike.status+item.status}}% {{$t(`modular.status`)}}</span>
          </div>
        </el-radio>
      </div>
    </div>
    <!-- 部件 -->
    <div class="parts">
      <div class="part" v-if="strike">{{$t("zaw.strike")}}: {{$t(`messages.${strike.name}`)}}</div><!--
   --><div class="part" v-if="grip">{{$t("zaw.grip")}}: {{$t(`messages.${grip.name}`)}}</div><!--
   --><div class="part" v-if="links">{{$t("zaw.links")}}: {{$t(`messages.${links.name}`)}}</div>
    </div>
    <!-- 预览 -->
    <div class="preview" v-if="strike">
      <div class="prop">{{$t("modular.damage")}}: {{+zaw.panelDamage.toFixed(1)}}</div><!--
   --><div class="prop">{{$t("modular.slideDamage")}}: {{+zaw.slideDmg.toFixed(1)}}</div><!--
   --><div class="prop" v-for="dmg in zaw.dmg" :key="dmg[0]"><WfIcon :type="dmg[0].toLowerCase()"/> {{$t(`elements.${dmg[0]}`)}}: {{dmg[1]}}</div><!--
   --><div class="prop" v-if="zaw.range">{{$t("modular.range")}}: {{+(zaw.range[0]+zaw.range[1]).toFixed(1)}}m</div><!--
   --><div class="prop">{{$t("modular.fireRate")}}: {{zaw.fireRate}}</div><!--
   --><div class="prop">{{$t("modular.critDamage")}}: {{zaw.critMul}}x</div><!--
   --><div class="prop">{{$t("modular.critChance")}}: {{(zaw.critChance*100).toFixed()}}%</div><!--
   --><div class="prop">{{$t("modular.status")}}: {{(zaw.status*100).toFixed()}}%</div>
    </div>
    <el-button class="stepctl" :disabled="part === 0" @click="part = part > 0 ? part - 1 : 0">{{$t("modular.lastStep")}}</el-button>
    <el-button class="stepctl" :disabled="part === 0 && !strike || part === 1 && !grip || part === 2 && !links" @click="part = part < 2 ? part + 1 : (finish(),2)">{{part === 2 ? $t("modular.finish") : $t("modular.nextStep")}}</el-button>
  </div>
</template>
<script lang="ts">
import { Vue, Component, Watch, Prop } from "vue-property-decorator";
import { ZawStrikeData, ZawGripData, ZawLinksData, ZawStrike, ZawGrip, ZawLinks, Zaw } from "@/warframe/codex";
import "@/less/modular.less";

@Component
export default class extends Vue {
  part = 0;
  strikeList = ZawStrikeData;
  gripList = ZawGripData;
  linksList = ZawLinksData;
  strike: ZawStrike = null;
  grip: ZawGrip = null;
  links: ZawLinks = null;

  get zaw() {
    return new Zaw(this.strike, this.grip, this.links);
  }

  finish() {
    this.$emit("finish", this.zaw);
  }

  gripDmg(grip: ZawGrip) {
    let modify = grip.twoHand ? this.strike.twoHand : this.strike.oneHand;
    return ((72 + this.strike.dmg + grip.dmg + (this.links ? this.links.dmg : 0)) * modify.dmg).toFixed();
  }

  linkDmg(link: ZawLinks) {
    let modify = this.grip.twoHand ? this.strike.twoHand : this.strike.oneHand;
    return ((72 + this.strike.dmg + this.grip.dmg + link.dmg) * modify.dmg).toFixed();
  }
}
</script>
